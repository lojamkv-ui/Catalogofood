# 🍔 MVKV Food — Cardápio Online

Aplicativo web (HTML puro, sem dependências) de **catálogo / cardápio digital** para rede de fast food, inspirado em plataformas modernas como [Anota.ai](https://anota.ai). Design mobile-first, visual limpo e foco em conversão: montar sacola → cadastrar entrega → confirmar → acompanhar.

> Projeto de demonstração: toda a lógica roda no navegador (vanilla JS + localStorage). Nenhum dado sai do dispositivo.

## ✨ Funcionalidades

| Área | O que faz |
|---|---|
| **Cardápio digital** | 22 itens em 7 categorias (combos, burgers, pizzas, acompanhamentos, bebidas, sobremesas), busca em tempo real, filtros por categoria, tags de destaque. |
| **Login de usuário** | Modal com abas **Cliente / Loja / Entregador**. Cliente entra com nome + WhatsApp (máscara de telefone); loja usa o acesso demo `loja@mvcfood.com` / `mvkv123` e entregador usa `entregador@mkvfood.com` / `mvkv123`. Sessões persistidas em `localStorage`, avatar, menu de conta e logout. |
| **Área da loja** | Painel em `#/loja` com estatísticas, filtros de pedidos, avanço manual de status, endereço/cliente e botão de WhatsApp. Ao entrar como loja, a simulação automática de status fica pausada. |
| **Área do entregador** | Painel em `#/entregador` com as entregas atribuídas ao entregador logado: cliente, itens, total, WhatsApp, endereço, **ETA até a loja** e o fluxo manual em 5 etapas — **Aguardando coleta → Indo para a loja → Pedido coletado → A caminho do cliente → Entregue**. Ao entrar, a simulação automática fica pausada e o avanço é feito etapa a etapa. |
| **Distribuição automática** | A cada novo pedido de entrega, `assignCourier()` calcula o tempo estimado total de cada entregador (ETA até a loja + trecho loja→cliente + penalidade por entregas em aberto) e atribui o pedido ao entregador com o **menor tempo estimado total**. |
| **Entrega com localização** | Botão **“Usar minha localização (GPS)”** (`navigator.geolocation`) → calcula distância até a loja (Haversine) e ajusta tempo de entrega e status. Fallback com formulário manual (CEP, rua, número, bairro, referência, observações). Alternância **Entrega 🛵 / Retirada 🏪**. |
| **Sacola** | Drawer lateral com stepper de quantidade, barra de progresso para **frete grátis** (acima de R$ 89), resumo com subtotal/taxa/total. |
| **Pagamento** | Pix 💠, Cartão 💳 ou Dinheiro 💵 (com campo de troco). |
| **Confirmação via WhatsApp** | Ao confirmar, abre `wa.me` com mensagem formatada (itens, total, endereço, coordenadas GPS, pagamento e cliente). |
| **Acompanhamento em tempo real** | Linha do tempo com 4 estados (Recebido → Em preparo → **A caminho** → Entregue), ETA dinâmico e avanço automático (simulação). |
| **Notificações** | Centro de notificações no sino 🔔 + **Web Notifications** do navegador. Aviso em destaque (toast + notificação) quando o pedido **sai para entrega**. |
| **Chat com o cliente** | Widget de chat flutuante com atendente simulado (responde status do pedido, tempo de entrega, pagamentos, frete) e respostas rápidas. |

## 🚀 Como rodar

É só abrir o `index.html` no navegador — ou servir a pasta:

```bash
# opção 1: abrir direto
open index.html

# opção 2: servidor local
python3 -m http.server 8080
# → http://localhost:8080
```

> **Nota:** geolocalização e Web Notifications exigem contexto seguro (`https` ou `localhost`).

## 🏪 Área da loja (demo)

Abra **Entrar → Loja** para acessar o painel interno. O acesso demonstrativo é:

- E-mail: `loja@mvcfood.com`
- Senha: `mvkv123`

O painel fica em `#/loja` e exibe estatísticas, pedidos, dados do cliente, endereço de entrega, WhatsApp e o botão **“Avançar status”**. No modo loja, o avanço automático usado na simulação do cliente fica pausado para que cada status seja controlado manualmente.

## 🛵 Área do entregador (demo)

Abra **Entrar → Entregador** para acessar o painel do entregador. O acesso demonstrativo é:

- E-mail: `entregador@mkvfood.com`
- Senha: `mvkv123`

O painel fica em `#/entregador` e lista as entregas atribuídas ao entregador logado (o usuário demo é o entregador **Carlos**), com cliente, itens, total, WhatsApp, endereço, **ETA até a loja** e um fluxo manual em 5 etapas:

1. **Aguardando coleta** — pedido pronto/entrando na fila na loja
2. **Indo para a loja** — entregador a caminho da coleta
3. **Pedido coletado** — pedido recolhido na loja
4. **A caminho do cliente** — em rota de entrega (o cliente vê “A caminho”)
5. **Entregue** — entrega concluída (o cliente vê “Entregue”)

No modo entregador, a simulação automática de status fica pausada: cada etapa é avançada manualmente no painel, e as visões do cliente e da loja acompanham o progresso. O painel da loja (`#/loja`) também mostra em qual entregador cada pedido de entrega foi distribuído.

### ⚙️ Regra de distribuição (automática)

Ao confirmar um pedido com **entrega**, a função `assignCourier()` pontua cada entregador de `CONFIG.couriers` assim:

```
ETA total = (base 5 min + distância posição do entregador → loja × 1,6 min/km)
          + distância loja → cliente × 1,6 min/km  (2,5 km quando o cliente não passou o GPS)
          + 6 min × (entregas em aberto daquele entregador)
```

O pedido é atribuído ao entregador com o **menor tempo estimado total** (empates decididos por quem tem menos entregas em aberto). Pedidos de **retirada na loja** não recebem entregador.

## ⚙️ Configuração

Tudo está centralizado no objeto `CONFIG` no topo do `<script>` do `index.html`:

```js
const CONFIG = {
  whatsapp: '5511987654321',            // ← número do restaurante (DDI 55 + DDD)
  store: { lat: -23.5505, lng: -46.6333, address: '...' }, // ← coordenadas da loja
  deliveryFee: 5.90,                    // ← taxa de entrega
  freeDeliveryAbove: 89.00,             // ← valor do frete grátis
  baseEtaMin: 28, etaPerKm: 1.6,        // ← estimativa de tempo
  statusAt: [8000, 30000, 75000],       // ← timing da simulação de status (ms)
  storeLogin: { email, password },      // ← acesso demo da loja
  deliveryLogin: { email, password },   // ← acesso demo do entregador
  couriers: [ { id, name, pos }, ... ], // ← frota (posições p/ cálculo de ETA)
  courierBaseMin: 5,                    // ← minutos base do entregador até sair
  courierPenaltyMin: 6,                 // ← penalidade por entrega em aberto
  defaultLegKm: 2.5,                    // ← trecho médio sem GPS do cliente
};
```

O cardápio é o array `MENU` no mesmo arquivo — adicione/remova itens editando esse JSON.

## 🧩 Estrutura

```
Catalogofood/
├── index.html    # app completo (HTML + CSS + JS inline, zero dependências)
└── README.md
```

## 🔭 O que é simulado (auditoria)

Como o app roda 100% no front-end, os pontos abaixo são **simulações de demonstração** e devem ser integrados a um backend real em produção:

- **Login** — apenas `localStorage` (sem senha). Produção: autenticação real (ex.: Google/Firebase) + criptografia de PII.
- **WhatsApp** — usa link `wa.me` com a mensagem pronta (não há envio automático; o cliente aperta “enviar”).
- **Pagamento** — nenhuma cobrança real. Produção: gateway (Pix via PSP, cartão via Adyen/Stripe/Asaas).
- **Status do pedido** — avança por timer. Produção: webhooks do sistema da cozinha/entregadores.
- **Chat** — bot com respostas por palavras-chave. Produção: API do WhatsApp Business + CRM.
- **Entrega** — taxa/ETA calculados por distância até as coordenadas fixas da loja; a distribuição para a frota é simulada pelo menor tempo estimado total (ver regra na seção do entregador), com posições fixas dos entregadores. Produção: sistema de despacho com GPS real da frota.

## 🔒 Privacidade

- Dados (nome, telefone, endereço, pedidos, chat) ficam **somente no navegador** do visitante.
- Ao confirmar o pedido, o endereço e as coordenadas GPS são incluídos na mensagem de WhatsApp para o restaurante — comportamento esperado para entrega, documentado aqui e no app.

---

© 2026 MVKV Food · demo de catálogo digital
