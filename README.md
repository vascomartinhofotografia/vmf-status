# Status Page — Vasco Martinho Fotografia

Página estática mostrada quando os serviços online estão com problemas.

**Auto-contida**: HTML único com CSS + JS inline. Sem build, sem deps.

## Onde hospedar (escolher uma)

A regra é: **hospedar FORA do Firebase Hosting**, para que se Firebase cair,
esta página ainda apareça aos clientes. Opções de mais simples → mais robusto:

### Opção A — QNAP (mais rápido, depende do NAS estar ligado)

1. Copiar `index.html` para a partilha `Web` do QNAP via SMB ou:
   ```bash
   scp index.html qnap:/share/Web/status/index.html
   ```
2. Activar Web Server qpkg no QNAP (App Center → Web Server)
3. Configurar `status.vascomartinhofotografia.com` para apontar para o IP público do QNAP (precisa de DDNS + port forward 80/443 no router)
4. Configurar HTTPS no QTS (Control Panel → Network & File Services → Web Server → SSL)

**Caveat:** se o QNAP estiver offline (ex.: corte de luz em casa), a status page também cai.

### Opção B — Cloudflare Pages (recomendada, grátis, CDN global) ⭐ ESCOLHIDA

Vantagens: independente de Firebase, Bluehost e do QNAP. CDN global. Grátis até 500 builds/mês.
Já temos `_headers` configurado (security headers + cache).

#### Passo 1 — Criar projecto no Cloudflare Pages

1. Login em https://dash.cloudflare.com → **Workers & Pages** → **Create application** → **Pages**
2. Escolher **Upload assets** (forma mais simples, sem precisar de Git):
   - Project name: `vmf-status`
   - Production branch: `main` (irrelevante para upload direct)
3. Drag-and-drop da pasta `status-page/` inteira
4. **Deploy site** — fica disponível em `https://vmf-status.pages.dev`

**Alternativa**: se quiseres deploy automático via Git, podes ligar o repo
GitHub/GitLab — Cloudflare reconstrói sempre que pushes alterações em
`status-page/`. Config: build command vazio, output dir `status-page`.

#### Passo 2 — Custom domain

1. No projecto Pages criado → **Custom domains** → **Set up a custom domain**
2. Domain: `status.vascomartinhofotografia.com`
3. Cloudflare diz qual o registo DNS a adicionar — se o teu domínio **já estiver
   gerido por Cloudflare**, faz automaticamente. Se estiver no Bluehost, mostra:
   - Tipo: CNAME
   - Nome: `status`
   - Target: `vmf-status.pages.dev`
4. Cria o registo no painel Bluehost (cPanel → Zone Editor → Add CNAME)
5. Volta ao Cloudflare → **Verify** (pode demorar 5-30min a propagar)
6. SSL: Cloudflare auto-gera certificado Let's Encrypt — não precisas fazer nada

#### Passo 3 — Verificar

```bash
# Resolução DNS
nslookup status.vascomartinhofotografia.com

# HTTP
curl -I https://status.vascomartinhofotografia.com
# Esperado: HTTP/2 200 + headers de security

# Browser → https://status.vascomartinhofotografia.com
# Deve mostrar a page com 4 checks a actualizar
```

#### Updates futuros

Para alterar a página (adicionar serviços, mudar contactos, etc.):
- **Se usaste upload direct**: edita local + drag-and-drop nova versão no Cloudflare
- **Se ligaste Git**: edita + commit + push → auto-deploy

Cloudflare guarda histórico de deploys → rollback com 1 clique se algo correr mal.

### Opção C — GitHub Pages (grátis)

1. Criar repo público `vmf-status` no GitHub
2. Copiar `index.html` para a raiz
3. Settings → Pages → activar → branch `main`
4. CNAME para `status.vascomartinhofotografia.com`

## Configuração DNS

Depois de escolher hosting, configurar (no painel DNS do registar — provavelmente Bluehost):

| Tipo | Nome | Valor |
|---|---|---|
| CNAME | `status` | (depende do provider escolhido) |

Para Cloudflare Pages: `vmf-status.pages.dev`
Para GitHub Pages: `<username>.github.io`
Para QNAP: registo A para o IP público (precisa DDNS)

## Onde aparece o link

Para os clientes verem a status page quando algo falha:
- **Toast de erro no Autorizacao.jsx**: mencionar `status.vascomartinhofotografia.com` quando rede falha
- **Email de notificação de outage** (alertas automáticos podem incluir o URL)
- **Bookmark do admin** para verificar rapidamente

## Manutenção

A página é estática mas faz **HEAD probes em JavaScript** aos 4 serviços principais.
Não precisa de atualização — auto-actualiza a cada 30s.

Quando o serviço estiver OK, a página mostra:
- 🟢 Verde — < 1500ms
- 🟡 Amarelo — 1500-5000ms
- 🔴 Vermelho — sem resposta / timeout

Se quiseres adicionar/remover serviços, edita o array `services` no script.
