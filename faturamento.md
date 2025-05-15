# 📊 Análise da Função de Faturamento Whitelabel

## 🎯 Objetivo
Calcular e retornar dados de faturamento para o dashboard administrativo, incluindo entradas, taxas e valores líquidos.

## 💰 Componentes do Faturamento

### 📥 Entradas
| Componente | Descrição |
|------------|-----------|
| `entradasTransacoes` | Taxas das transações pagas |
| `entradasSaque` | Valores de saques |
| `entradasAntecipacao` | Taxas de antecipação |
| `entradasPreChargeback` | Valores de prechargeback |

### 📤 Saídas
| Componente | Descrição |
|------------|-----------|
| `estornos` | Valores estornados |
| `chargeback` | Valores de chargeback |

### 💸 Taxas
| Tipo | Descrição |
|------|-----------|
| `taxasDeAdquirente` | Taxas da adquirente |
| `taxasBaas` | Taxas do serviço BaaS |
| `taxasGetway` | Taxas do gateway |

## 📊 Cálculos Principais

### Faturamento Total
```typescript
faturamentoTotal = entradasSaque + 
                   entradasTransacoes + 
                   entradasAntecipacao + 
                   entradasPreChargeback
```

### Valor Líquido
```typescript
valorLiquido = faturamentoTotal - 
               (estornos + 
                chargeback + 
                taxasDeAdquirente + 
                taxasBaas + 
                taxasGetway)
```

## 🔄 Processo de Cálculo

1. **Cálculo de Entradas**
   - Taxa de intermediação das transações pagas 
   - Taxa de saques
   - Taxa de antecipações
   - Prechargebacks

2. **Cálculo de Saídas**
   - Estornos
   - Chargebacks

3. **Cálculo de Taxas**
   - Taxas cobradas pela adquirente
   - Taxas cobradas pelo BaaS
   - Taxas cobradas pela whitelabel

4. **Cálculo Final**
   - Faturamento total
   - Valor líquido
