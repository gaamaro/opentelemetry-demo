.PHONY: get-secrets setup-ssh deploy check-health

get-secrets:
	@echo "Obtendo secrets do Vault..."
	@vault login ${VAULT_TOKEN}
	@vault kv get -field=ip kv/data/ec2-ip > .ec2_ip
	@vault kv get -field=private_key kv/data/ssh-default > .ssh_key
	@chmod 600 .ssh_key
	@vault kv get -field=DT_ENDPOINT kv/data/dynatrace > .dt_endpoint

setup-ssh:
	@echo "Configurando SSH..."
	@mkdir -p ~/.ssh
	@mv .ssh_key ~/.ssh/ec2_private_key
	@echo "Host ec2\n\tHostName $$(cat .ec2_ip)\n\tUser ec2-user\n\tIdentityFile ~/.ssh/ec2_private_key\n\tStrictHostKeyChecking no" > ~/.ssh/config

deploy:
	@echo "Deployando aplicação..."
	@export DT_ENDPOINT=$$(cat .dt_endpoint) && \
	ssh ec2 'cd /repo/opentelemetry-demo && \
	docker compose up --force-recreate --remove-orphans --detach'

check-health:
	@echo "Verificando saúde dos containers..."
	@ssh ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "up"'
	@ssh ec2 'docker ps --format "{{.Names}} - {{.Status}}" | grep -i "unhealthy" || echo "Todos os containers estão saudáveis"'

deploy-all: get-secrets setup-ssh deploy check-health 