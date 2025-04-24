.PHONY: deploy-all deploy check-health

deploy:
	@echo "🚀 Iniciando deploy na EC2 remota..."
	@ssh ec2 'cd /repo/opentelemetry-demo && \
		export $$(cat dynatrace.env | xargs) && \
		docker compose pull && \
		docker compose up --force-recreate --remove-orphans --detach'

check-health:
	@echo "🔍 Verificando saúde dos containers na EC2..."
	@echo "⏳ Aguardando inicialização dos containers..."
	@for i in $$(seq 1 6); do \
		echo "Tentativa $$i de 6..."; \
		sleep 10; \
		CONTAINERS_UP=$$(ssh ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "up" | wc -l'); \
		CONTAINERS_TOTAL=$$(ssh ec2 'docker ps --format "{{.Names}}" | wc -l'); \
		if [ "$$CONTAINERS_UP" = "$$CONTAINERS_TOTAL" ]; then \
			echo "✅ Todos os containers estão em execução!"; \
			break; \
		fi; \
		if [ "$$i" = "6" ]; then \
			echo "❌ Timeout: Nem todos os containers iniciaram após 60 segundos"; \
			ssh ec2 'docker ps --format "{{.Names}} - {{.Status}}"'; \
			exit 1; \
		fi; \
	done
	@echo "🔍 Verificando status de saúde..."
	@UNHEALTHY=$$(ssh ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "unhealthy" | wc -l'); \
	if [ "$$UNHEALTHY" -gt 0 ]; then \
		echo "⚠️ Containers unhealthy encontrados:"; \
		ssh ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "unhealthy"'; \
		exit 1; \
	else \
		echo "✅ Todos os containers estão saudáveis!"; \
	fi

deploy-all: deploy check-health
	@echo "✨ Deploy finalizado com sucesso!" 