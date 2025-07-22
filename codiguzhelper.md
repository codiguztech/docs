# CodiguzHelper - Documentação de Integração

## Visão Geral

O CodiguzHelper é uma biblioteca JavaScript para criptografia segura de dados de cartão de crédito em integrações de pagamento. Esta documentação descreve como implementar a biblioteca em sua aplicação.

## Pré-requisitos

- Script do CodiguzHelper carregado em sua página
- Public Key (Company ID) fornecida pelo painel Luna
- Formulário HTML com campos de cartão de crédito

## Guia de Implementação

### 1. Inicialização

Após carregar o script do CodiguzHelper, execute os seguintes passos:

```javascript
// Obter o nome da variável do gateway dinamicamente
let variableName = window.CodiguzHelper.getJsVariableName();

// Configurar a Public Key (Company ID)
window[variableName].setPublicKey('1b4eb306-78af-4a3e-815b-85a8d7c81427');
```

**Nota:** A Public Key deve ser obtida no painel Luna e é específica para cada empresa.

### 2. Preparação dos Dados do Cartão

Colete os dados do cartão do formulário e estruture-os no formato esperado:

```javascript
const cardData = {
    holderName: "Nome do Titular",
    number: document.getElementById('cardNumber').value.replace(/\s/g, ''),
    cvv: document.getElementById('cardCvv').value,
    expMonth: document.getElementById('cardExp').value.split('/')[0],
    expYear: document.getElementById('cardExp').value.split('/')[1]
};
```

**Campos obrigatórios:**
- `holderName`: Nome do titular do cartão
- `number`: Número do cartão (sem espaços)
- `cvv`: Código de segurança
- `expMonth`: Mês de expiração (formato: MM)
- `expYear`: Ano de expiração (formato: YY ou YYYY)

### 3. Criptografia dos Dados

Utilize o método `encrypt` para criptografar os dados do cartão:

```javascript
const encryptedData = await window[variableName].encrypt(cardData);
```

### 4. Estrutura do Retorno

O método `encrypt` retorna um objeto com a seguinte estrutura:

```json
{
    "token_id": "72d71479-11c5-4d1a-86b7-375fd75e05f4",
    "id": "72d71479-11c5-4d1a-86b7-375fd75e05f4"
}
```

## Criação de Transação

Utilize o `id` retornado na criação da transação. Exemplo de payload completo:

```json
{
    "customer": {
        "name": "Nome do Cliente",
        "email": "cliente@example.com",
        "phone": "11999999999",
        "document": {
            "number": "12345678900",
            "type": "CPF"
        }
    },
    "shipping": {
        "address": {
            "street": "Rua Exemplo",
            "streetNumber": "123",
            "complement": "Apto 45",
            "zipCode": "01234567",
            "neighborhood": "Centro",
            "city": "São Paulo",
            "state": "SP",
            "country": "BR"
        }
    },
    "paymentMethod": "CARD",
    "card": {
        "id": "72d71479-11c5-4d1a-86b7-375fd75e05f4",
        "number": "4000000000000010",
        "holderName": "Nome do Titular",
        "expirationMonth": 12,
        "expirationYear": 2025,
        "cvv": "123"
    },
    "installments": 1,
    "items": [
        {
            "title": "Produto 01",
            "unitPrice": 5000,
            "quantity": 1,
            "externalRef": "PROD001"
        }
    ],
    "amount": 5000,
    "postbackUrl": "https://suaapi.com/webhook",
    "metadata": "{\"pedido_id\":\"12345\"}",
    "ip": "192.168.1.1",
    "description": "Descrição da compra",
    "fingerPrint": "identificador-unico"
}
```

### Campos Importantes

- **card.id**: Use o ID retornado pela criptografia
- **amount**: Valor total em centavos (ex: 5000 = R$ 50,00)
- **installments**: Número de parcelas
- **postbackUrl**: URL para receber notificações de status
- **metadata**: Dados extras em formato JSON string
