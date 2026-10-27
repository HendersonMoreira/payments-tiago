# Checklist rapido Mercado Pago (1 minuto)

Use este fluxo antes de testar checkout no TioTV.

## 1) No painel do Mercado Pago

1. Entrar no Mercado Pago com a mesma conta do access token usado no backend.
2. Ir em `Seu negocio` > `Configuracoes` > `Dados da conta`.
3. Confirmar e-mail da conta ate ficar marcado como confirmado.
4. Ir em `Seu negocio` > `Cobrar com Mercado Pago` (ou `Checkout`) e concluir pendencias de cadastro/faturamento.
5. Validar que a conta pode receber cobrancas (`billing allow`).

## 2) Validar no backend (payments-server)

Com o servidor rodando, chamar:

- `GET /api/payments/mercadopago/account-status`

Exemplo local:

```powershell
Invoke-RestMethod -Uri "http://localhost:8787/api/payments/mercadopago/account-status" -Method Get | ConvertTo-Json -Depth 10
```

Resultado esperado para liberar teste de checkout:

- `checks.confirmedEmail.value = true`
- `checks.billingAllow.value = true`
- `readyForCheckout = true`

Se `billingAllow` vier `null`, o schema da conta pode nao expor esse campo pela API. Nesse caso, use o painel do MP como fonte oficial e reteste o checkout.

## 3) Teste final

1. Chamar `POST /api/payments/pagbank/checkout`.
2. Abrir `paymentUrl` retornada.
3. Finalizar pagamento de teste com usuario/cartao de teste do MP.
4. Chamar `POST /api/payments/pagbank/verify` para confirmar ativacao premium.

## Checkout da loja de tenis (ambiente de teste)

A loja usa a rota `POST /api/store/checkout` e o Checkout Pro hospedado pelo Mercado Pago. O servidor valida os IDs, tamanhos e quantidades com o catalogo local, sem confiar nos precos enviados pelo navegador.

Antes do teste local:

1. Configure um access token de teste (`TEST-...`) em `MERCADOPAGO_ACCESS_TOKEN`. O modo padrao da loja e `STORE_CHECKOUT_MODE=test` e recusa tokens de producao.
2. Configure `STORE_FRONTEND_URL=http://localhost:5173` no servidor.
3. Configure `VITE_PAYMENTS_API_URL=http://localhost:8787` no `.env` da loja; use o `.env.example` como referencia.
4. Inicie o servidor de pagamentos e o Vite em terminais separados.
5. Selecione produtos, preencha os dados e escolha Pix. A loja deve redirecionar ao checkout de teste do Mercado Pago.

Para producao, configure `STORE_CHECKOUT_MODE=live`, um token de producao no servidor, `STORE_FRONTEND_URL` com o dominio HTTPS da loja e `VITE_PAYMENTS_API_URL` com a URL HTTPS do servidor. Nao use tokens reais para validar testes.

O resultado e consultado em `GET /api/store/payments/:paymentId`. A loja so mostra o pagamento como confirmado depois de validar o status com o Mercado Pago; os parametros de retorno do navegador, sozinhos, nao confirmam pagamento.
