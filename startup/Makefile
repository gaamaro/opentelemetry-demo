.PHONY: deploy-all deploy check-health

deploy:
	@echo "🚀 Iniciando deploy na EC2 remota..."
	@ssh ec2 'cd /repo/opentelemetry-demo && \
		export $$(cat dynatrace.env | xargs) && \
		docker compose pull && \
		docker compose up --force-recreate --remove-orphans --detach'

check-health:
	@echo "🔍 Verificando saúde dos containers na EC2..."
	@sleep 30  # Aguarda 30 segundos para os containers inicializarem
	@ssh ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "up"' || (echo "❌ Nenhum container em execução!" && exit 1)
	@ssh ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "unhealthy"' && echo "⚠️ Containers unhealthy encontrados!" || echo "✅ Todos os containers estão saudáveis"

deploy-all: deploy check-health
	@echo "✨ Deploy finalizado com sucesso!" 