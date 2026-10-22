# Streaming Music

Projeto CRUD Laravel para a atividade **T2 - Projeto CRUD Laravel**.

Tema implementado: **Sistema de Streaming de Musica**, com cadastro de musicas e playlists relacionadas.

## Requisitos atendidos

- Laravel com rotas, controllers, models, migrations e views Blade.
- Tabela principal `musicas` com `titulo`, `artista`, `album`, `genero`, `duracao` e `capa`.
- Tabela secundaria `playlists` com `nome_playlist`, `musica_id`, `usuario`, `data_criacao`, `descricao` e `status`.
- Relacionamento obrigatório: uma playlist pertence a uma musica.
- CRUD completo para musicas e playlists.
- Upload de imagem para capa da musica.
- Campo de categoria/status: `genero` em musicas e `status` em playlists.
- Autenticacao Laravel feita com `Auth`, `Hash` e model `User`.
- Visitantes podem listar e consultar; criar, editar e excluir exige login.
- Layout proprio com Bootstrap e CSS customizado.

## Configuracao com Docker

O projeto ja vem configurado para usar MySQL via Docker na porta `3307`:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3307
DB_DATABASE=streaming
DB_USERNAME=root
DB_PASSWORD=streaming_secret
```

Suba o banco:

```bash
docker compose up -d
```

Depois rode:

```bash
php artisan migrate --seed
php artisan storage:link
php artisan serve
```

Se preferir usar um MySQL local, altere `DB_PORT`, `DB_USERNAME` e `DB_PASSWORD` no `.env`.

## Deploy no AWS Learner Lab (EC2) com GitHub Actions

O arquivo `docker-compose.prod.yml` publica a aplicacao na porta 80 e mantem o
MySQL e os uploads em volumes persistentes.

Na instancia Amazon Linux 2023, instale Docker, Git e o plugin Compose:

```bash
sudo dnf update -y
sudo dnf install -y docker git
sudo systemctl enable --now docker
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/download/v5.6.0/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
sudo docker compose version
```

Depois clone o repositorio e suba a aplicacao:

```bash
git clone https://github.com/pedro010306santo-byte/Streaming.git
cd Streaming
cat > .env <<'EOF'
APP_URL=http://IP_PUBLICO_DA_EC2
DB_PASSWORD=COLOQUE_UMA_SENHA_FORTE_AQUI
MYSQL_ROOT_PASSWORD=COLOQUE_OUTRA_SENHA_FORTE_AQUI
RUN_SEEDER=false
EOF
sudo docker compose -f docker-compose.prod.yml up -d --build
```

Troque o IP e as duas senhas antes de executar o Compose. O arquivo `.env` fica
somente na EC2 e nao deve ser enviado ao GitHub. Para carregar os dados de
demonstracao uma unica vez, execute:

```bash
sudo docker compose -f docker-compose.prod.yml exec app php artisan db:seed --force
```

### Configurar o GitHub Actions

No repositorio, abra **Settings > Secrets and variables > Actions** e crie estes
repository secrets:

- `EC2_HOST`: IP publico ou DNS publico da instancia.
- `EC2_USER`: `ec2-user` no Amazon Linux 2023.
- `EC2_SSH_KEY`: conteudo completo da chave privada `.pem` da instancia.

O Security Group precisa aceitar TCP 22 (SSH) a partir dos runners do GitHub
durante o deploy e TCP 80 (HTTP) para os usuarios. A porta 3306 nao deve ser
publicada.

O workflow `.github/workflows/deploy-aws.yml` executa automaticamente a cada
push na branch `main`. Tambem e possivel inicia-lo manualmente pela aba
**Actions > Deploy na AWS EC2 > Run workflow**.

Acesse `http://IP_PUBLICO_DA_EC2`. O Security Group da instancia precisa liberar
TCP 80 (HTTP). Nao libere a porta 3306: o MySQL fica acessivel somente pela rede
interna do Docker.

Para conferir o deploy:

```bash
sudo docker compose -f docker-compose.prod.yml ps
sudo docker compose -f docker-compose.prod.yml logs --tail=100 app
```

Para aplicar uma atualizacao enviada ao GitHub:

```bash
git pull
sudo docker compose -f docker-compose.prod.yml up -d --build
```

As senhas do banco sao obrigatorias no arquivo `.env` da EC2. O deploy falha de
forma segura se elas nao estiverem configuradas.

## Usuario de teste (somente ao executar o seeder)

O seeder cria um usuario administrador:

- E-mail: `admin@streaming.test`
- Senha: `123456`

## Observacoes para apresentacao

O projeto segue a proposta das aulas e do curso do Matheus Battisti como referencia de CRUD Laravel, mas usa identidade visual propria para cumprir o criterio de layout original.
