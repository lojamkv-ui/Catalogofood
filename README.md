# 🍔 MVKV Food — Cardápio Online

Aplicativo web (HTML puro, sem dependências) de **catálogo / cardápio digital** para rede de fast food, inspirado em plataformas modernas como [Anota.ai](https://anota.ai). Design mobile-first, visual limpo e foco em conversão: montar sacola → cadastrar entrega → confirmar → acompanhar.

> Projeto de demonstração: toda a lógica roda no navegador (vanilla JS + localStorage). Nenhum dado sai do dispositivo.

## ✨ Funcionalidades

| Área | O que faz |
|---|---|
| **Cardápio digital** | 22 itens em 7 categorias (combos, burgers, pizzas, acompanhamentos, bebidas, sobremesas), busca em tempo real, filtros por categoria, tags de destaque. |
| **Login de usuário** | Modal com nome + WhatsApp (máscara de telefone). Sessão persistida em `localStorage`, avatar com iniciais, menu de conta e logout. |
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
- **Entrega** — taxa/ETA calculados por distância até as coordenadas fixas da loja.

## 🔒 Privacidade

- Dados (nome, telefone, endereço, pedidos, chat) ficam **somente no navegador** do visitante.
- Ao confirmar o pedido, o endereço e as coordenadas GPS são incluídos na mensagem de WhatsApp para o restaurante — comportamento esperado para entrega, documentado aqui e no app.

---

© 2026 MVKV Food · demo de catálogo digital
