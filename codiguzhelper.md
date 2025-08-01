# Guia de Implementação

## Carregamento do Script

Adicione o script do gateway em sua página:

```html
<script src="https://api.dev.codiguz.com/functions/v1/js?v=vega"></script>
```


### 1. Inicialização

Após carregar o script, execute os seguintes passos:

```javascript
// Obter o nome da variável do gateway dinamicamente
const variableName = window.CodiguzHelper.getJsVariableName();

// Configurar a Public Key (Company ID)
await window[variableName].setPublicKey("sua-public-key-aqui");
```

**Nota:** A Public Key deve ser obtida no painel do gateway e é específica para cada empresa.

### Como Funciona Internamente

Quando você chama `setPublicKey()`, automaticamente acontece:

1. **Requisição para Configurações de Antifraude**

   ```
   GET https://api.dev.codiguz.com/functions/v1/antifraud?publicKey={sua-public-key}
   ```

2. **Resposta com Configurações**

   ```jsx JSON
   {
       "antifraud": "PAYZU",               // Tipo de antifraude ativo
       "antifraudToken": [...],            // Tokens de autenticação
       "threeDSSecurity": true,            // Se 3DS está ativo
       "threeDSSecurityType": "SCRIPT",    // Tipo: NONE, SCRIPT, IFRAME, REDIRECT
       "threeDSScriptType": "PAYZU",       // Provedor do 3DS
       "three3DSScriptUrl": "https://...", // URL do script 3DS
       "iframeUrl": "https://...",         // URL para modo iframe
       "hideCardForm": false               // Se deve ocultar form de cartão
   }
   ```

3. **Carregamento Automático de Scripts**

   * Se antifraude está ativo, carrega script do provedor (PayZu, ClearSale, etc.)
   * Se 3DS tipo SCRIPT está ativo, carrega script do 3DS
   * Scripts são carregados dinamicamente no DOM

4. **Coleta Automática de Fingerprint**
   * Cada provedor de antifraude coleta fingerprint de forma diferente
   * O fingerprint é armazenado em um campo hidden com ID `CodiguzSessionIdToken`
   * Exemplos:
     * **ClearSale**: Cria session ID e envia para device.clearsale.com.br
     * **Konduto**: Usa Visitor ID do K-Analytix
     * **Cybersource**: Gera session único e carrega tags.js
     * **MercadoPago**: Usa deviceId do SDK v2
     * **Payzu**: Gera UUID único para sessão

### 2) Preparação dos Dados do Cartão

Colete os dados do cartão do formulário e estruture-os no formato esperado:

```javascript
const cardData = {
    holderName: "João da Silva",
    number: "4111111111111111",
    cvv: "123",
    expMonth: "12",
    expYear: "2025",
    amount: 10000, // R$ 100,00 em centavos
    installments: 1,
};
```

**Campos obrigatórios:**

* `holderName`: Nome do titular do cartão
* `number`: Número do cartão (sem espaços)
* `cvv`: Código de segurança
* `expMonth`: Mês de expiração (formato: MM)
* `expYear`: Ano de expiração (formato: YYYY)
* `amount`: Valor em centavos
* `installments`: Número de parcelas (int)

### 3. Criptografia dos Dados

Utilize o método `encrypt` para criptografar os dados do cartão:

```javascript
try {
    const encryptedData = await window[variableName].encrypt(cardData);
    console.log("Token gerado:", encryptedData);
} catch (error) {
    console.error("Erro na criptografia:", error.message);
}
```

### Como o 3D Secure Funciona

Durante o processo de `encrypt()`, se o 3DS estiver ativo, o processo é automático:

#### Tipo SCRIPT (Exemplo: Payzu)

1. **Preparação Automática**

   ```javascript
   // Internamente, a biblioteca chama:
   await prepareThreeDS({
       amount: 10000,
       installments: 1,
       currency: "BRL",
   });
   ```

2. **Execução do Challenge**

   ```javascript
   // A biblioteca automaticamente:
   // - Monta dados completos (cartão + billing)
   // - Chama payzu3DS.checkout()
   // - Aguarda resposta do challenge
   ```

3. **Resposta do 3DS**
   * `success`: Autenticação aprovada
   * `failure`: Autenticação falhou
   * `unenrolled`: Cartão não inscrito no 3DS
   * `disabled`: 3DS desabilitado para este cartão
   * `error`: Erro no processo

### Dados Coletados Automaticamente

Durante o `encrypt()`, a biblioteca coleta automaticamente:

1. **Fingerprint do Dispositivo**

   * User Agent, linguagem, plataforma
   * Resolução de tela, timezone
   * Plugins instalados
   * Informações de rede

2. **Session ID do Antifraude**

   * Busca automaticamente no elemento `#CodiguzSessionIdToken`
   * Cada provedor popula este campo diferentemente

3. **Cookie de Identificação**
   * Cookie com ID único criptografado
   * Criado automaticamente na primeira visita

### 4) Estrutura do Retorno

O método `encrypt` retorna um objeto com a seguinte estrutura:

```json
{
    "token_id": "72d71479-11c5-4d1a-86b7-375fd75e05f4",
    "id": "72d71479-11c5-4d1a-86b7-375fd75e05f4"
}
```

### 5. Criação da Transação

Envie o token recebido no índice `"id"` dentro do objeto `"card"` no payload da criação de transação, cada token é único e só pode ser usado uma única vez:

```javascript
const transactionPayload = {
    card: {
        id: encryptedData.id, // Use o valor do campo "id" retornado
    },
    amount: 10000,
    // ... outros campos da transação
};
```

## Exemplo Completo

```javascript
// Função para processar pagamento
async function processPayment() {
    try {
        // 1. Obter nome da variável
        const variableName = window.CodiguzHelper.getJsVariableName();

        // 2. Configurar Public Key
        await window[variableName].setPublicKey("sua-public-key-aqui");

        // 3. Coletar dados do formulário
        const cardData = {
            holderName: document.getElementById("card-holder").value,
            number: document
                .getElementById("card-number")
                .value.replace(/\s/g, ""),
            cvv: document.getElementById("card-cvv").value,
            expMonth: document
                .getElementById("card-expiry")
                .value.split("/")[0],
            expYear: document.getElementById("card-expiry").value.split("/")[1],
            amount: parseInt(
                document.getElementById("total-amount").value,
            installments: parseInt(
                document.getElementById("installments").value
            ),
        };

        // 4. Criptografar dados
        const encryptedData = await window[variableName].encrypt(cardData);

        // 5. Enviar para backend e criar transação no gateway ...
    } catch (error) {
        console.error("Erro no processamento:", error);
    }
}
```

## Tratamento de Erros

O método `encrypt` pode lançar os seguintes erros:

* **"Public key não configurada"**: Chame `setPublicKey()` antes de `encrypt()`
* **"card.number é obrigatório"**: Número do cartão ausente
* **"card.holderName é inválido"**: Nome do titular ausente ou inválido
* **"card.expMonth é inválido"**: Mês de expiração inválido
* **"card.expYear é inválido"**: Ano de expiração inválido
* **"card.cvv é inválido"**: CVV ausente ou com tamanho incorreto

## Considerações Importantes

1. **HTTPS**: Em produção, sempre use HTTPS
2. **Validação**: Valide os dados antes de enviar para criptografia
3. **Token Único**: Cada token gerado é de uso único
4. **Timeout**: Tokens têm validade limitada, use-os rapidamente

## Fluxo Resumido

1. **setPublicKey()** → Busca configurações e carrega scripts automaticamente
2. **Scripts de antifraude** → Coletam fingerprint do dispositivo em background
3. **encrypt(cardData)** → Recebe todos os dados necessários via parâmetro cardData e inicia 3ds caso esteja ativo nas configurações do seller e retorna um id que deve ser passado dentro do objeto "card" no payload de criação de transação.
4. **Token retornado** → Use o campo `id` para criar a transação no backend
