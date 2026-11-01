# C.O.R.A

Central de Organização para Realocação e Acolhimento — mapeamento colaborativo de riscos em Itapecerica da Serra (SP).

Trabalho de conclusão de curso com duas interfaces no mesmo backend:

- **Site Django** em `http://127.0.0.1:8000`
- **App Flet** (cliente) que consome a **API REST** em `/api/`

## Requisitos

- Python 3.12+
- Git (o pacote `django-shadcn` é instalado a partir do GitHub)
- pip

## Como rodar localmente

```bash
git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
cd SEU_REPOSITORIO

python3 -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

pip install -r requirements.txt

cp .env.example .env
# edite DJANGO_SECRET_KEY no .env

python manage.py migrate
python manage.py createsuperuser
```

Em um terminal, suba a API/site:

```bash
python manage.py runserver
```

Em outro terminal, com o mesmo `venv` ativado:

```bash
python -m flet_app
```

O Flet chama `http://127.0.0.1:8000/api` por padrão. Para outra URL:

```bash
export CORA_API_URL=https://seu-dominio.com/api
python -m flet_app
```

## Contas

Não há usuários versionados (o SQLite fica fora do Git). Crie o seu:

- `python manage.py createsuperuser` — acesso administrativo (`is_staff`)
- cadastro pelo site ou pelo app — usuário comum

Marque `is_staff` no admin Django (`/admin/`) para o painel de relatórios e alertas.

## API

Autenticação: JWT (`Authorization: Bearer <access>`).

| Método | Caminho | Quem |
| --- | --- | --- |
| POST | `/api/auth/login/` | público |
| POST | `/api/auth/register/` | público |
| POST | `/api/auth/refresh/` | público |
| GET | `/api/me/` | autenticado |
| GET | `/api/meta/` | público |
| GET | `/api/cep/<cep>/` | público |
| POST | `/api/geocode/` | autenticado |
| GET | `/api/home/` | autenticado |
| GET/POST | `/api/incidents/` | autenticado |
| GET/PATCH | `/api/incidents/<id>/` | autenticado (dono ou staff) |
| GET | `/api/map/` | autenticado |
| GET | `/api/reports/` | staff |
| GET | `/api/alerts/` | staff |

Login:

```json
{ "email": "seu@email.com", "password": "senha" }
```

## Estrutura

```
accounts/          perfil (CPF, endereço, ViaCEP)
api/               REST + JWT
app/               settings, URLs, templates
authentication/    login e cadastro web
dashboards/        home, mapa, relatórios (site)
incidents/         denúncias e fotos
services/          e-mail de alerta, geocode, ViaCEP
flet_app/          cliente mobile/web Flet
static/            CSS
media/             uploads (não versionados)
```

## Enviar para o GitHub

Na pasta do projeto (já com este README e o `.gitignore`):

```bash
git init
git add .
git status          # confira: venv, .env e db.sqlite3 NÃO devem aparecer
git commit -m "Initial commit: C.O.R.A Django, API e app Flet"

git branch -M main
git remote add origin https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
git push -u origin main
```

Pela CLI do GitHub (`gh`):

```bash
gh repo create SEU_REPOSITORIO --private --source=. --remote=origin --push
```

Não versionar: `venv/`, `.env`, `db.sqlite3`, fotos em `media/`.

## Produção / loja de aplicativos

1. Hospede o Django com HTTPS.
2. Defina `DJANGO_DEBUG=False`, `DJANGO_SECRET_KEY` forte e `DJANGO_ALLOWED_HOSTS` com o domínio.
3. No app, use `CORA_API_URL=https://seu-dominio.com/api`.
4. O Flet na máquina de desenvolvimento continua apontando para `127.0.0.1` se a variável não for definida.
