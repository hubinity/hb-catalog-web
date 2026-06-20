# hb-catalog-web (Frontend) - Ecossistema Hubinity - Planned

> Parte integrante do ecossistema distribuído Hubinity.
> ⚠️ **Status atual: Planned** — código de implementação ainda não foi escrito. Este README descreve o papel arquitetural pretendido conforme PRD seção 4 e roadmap em `docs/phases/`.

---

## 💻 Visão Geral

- **O que faz:** Backoffice administrativo do catálogo HiBit. Permite listar e filtrar produtos, cadastrar e editar com upload de imagens, navegar a árvore de categorias e gerir estoque (saldo + histórico de movimentos).
- **Problema que resolve:** elimina a planilha de produtos/estoque; dá ao admin uma UI única para a fonte da verdade que alimenta tanto o backoffice quanto o totem da cafeteria.
- **Posicionamento no Ecossistema:** consumidor único e canônico do `hb-catalog-service`. É a UI administrativa do **source of truth** do catálogo.

## 🏗️ Papel na Arquitetura

- **Tipo de Componente:** Single-Page Application Angular 22 (standalone components + signals + control flow nativo `@if`/`@for`/`@switch`).
- **Responsabilidades Principais (planejadas):**
  - Listagem paginada e filtrável de produtos.
  - Form reativo de cadastro/edição com upload de imagens.
  - Visualização da árvore de categorias.
  - Tela de estoque com saldo corrente + histórico de movimentos.
  - Login OIDC contra Keycloak (PKCE).
- **Limites e Fronteiras (Boundaries):** não duplica regras de negócio do backend; toda validação de domínio acontece no `hb-catalog-service`.

## 🔗 Dependências e Comunicação (Planejadas)

### Serviços Internos da Hubinity

- **`hb-catalog-service`** — consumido via REST (`/api/v1/products`, `/api/v1/categories`, `/api/v1/stock/**`).
- **`platform-iam` (Keycloak)** — realm `hibit`, client público com PKCE; redirect URI `http://localhost:4200/*` em local e o domínio Vercel em staging/prod.
- **`@hubinity/tailwind-preset`** — pacote npm publicado no GitHub Packages com tokens de design (paleta HiBit, tipografia Inter, escala de spacing).

### Infraestrutura e Serviços Externos

- **Vercel** — hosting (plano Free, deploy automático a cada push em `main`, preview por PR).

## 🛠️ Tecnologias e Ferramentas (Stack Prevista)

| Camada | Tecnologia | Versão |
| :--- | :--- | :--- |
| Framework | Angular | 22 |
| Build | Angular CLI | compatível com 22 |
| Estilo | Tailwind CSS | 4 (via `@tailwindcss/postcss`) |
| Preset de tokens | `@hubinity/tailwind-preset` | última publicada |
| Primitives headless | `@spartan/ui` ou `ng-primitives` | última estável |
| Ícones | `lucide-angular` | última estável |
| Fonte | Inter via `@fontsource` | — |
| Auth | adapter Keycloak (PKCE) | — |
| E2E | Playwright | última estável |
| Hosting | Vercel | Free |

## 📐 Padrões de Projeto e Arquitetura do Código (Previstos)

- **Estilo Arquitetural:** SPA com componentes standalone, sem NgModules. Estado local com **signals**; estado compartilhado mínimo via services + signals.
- **Padrões Relevantes:**
  - **Reactive Forms** para cadastro/edição com validação cruzada client-side espelhando o backend.
  - **HTTP Interceptor** para anexar `Authorization: Bearer <jwt>` e fazer retry de 401 via refresh token.
  - **Route Guards** baseados em role `admin`.
  - Estilização **exclusivamente via Tailwind CSS 4** (sem SCSS por componente, sem CSS modules); densidade compacta (`text-sm` default) consistente com os outros backoffices.
  - Acessibilidade AA via primitives headless.

## 🗺️ Roadmap & Posição no Board

- **Fase do PRD:** Fase 1 — Catálogo (PRD seção 9).
- **Tasks no board:**
  - `1.9` — Bootstrap (Angular 22 + Tailwind 4 + preset + Keycloak + lucide).
  - `1.10` — Tela de listagem + filtro.
  - `1.11` — Form de cadastro/edição com upload de imagens.
  - `1.12` — Tela de categorias (árvore).
  - `1.13` — Tela de estoque (saldo + histórico).
  - `1.14` — E2E Playwright (cadastro → listagem → edição → baixa de estoque).
  - `1.15` — Dockerfile + workflow CI + deploy Vercel.
- **Dependências bloqueadoras:** `hb-catalog-service` com endpoints CRUD (Fase 1.5–1.7) e cliente Keycloak `hibit` provisionado.

## ⚙️ Variáveis de Ambiente (Previstas)

```bash
# Build-time / runtime injection via environment.ts
NG_APP_CATALOG_API_BASE_URL=https://hb-catalog-service.up.railway.app
NG_APP_KEYCLOAK_URL=https://iam.hubinity.app
NG_APP_KEYCLOAK_REALM=hibit
NG_APP_KEYCLOAK_CLIENT_ID=hb-catalog-web
```

## 🚀 Como Será Executado (Quando Implementado)

### Pré-requisitos

- Node.js 22 LTS
- Acesso ao GitHub Packages para baixar `@hubinity/tailwind-preset`
- Keycloak local rodando via `platform-infra`

### Execução (Será disponível após bootstrap da Fase 1.9)

```bash
npm ci
npm start         # dev server em http://localhost:4200
npm run build     # build prod (output: dist/)
npm test          # unit tests
npx playwright test   # E2E
```
