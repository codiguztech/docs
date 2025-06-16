# API Documentation - Ranking Endpoints

## Authentication

All endpoints require Basic Authentication:
- **Username**: `apikey`
- **Password**: Your company ID

Example:
```bash
curl -u "apikey:YOUR_COMPANY_ID" https://api.example.com/functions/v1/transactions/ranking
```

---

## Endpoints

### 1. Get Top 5 Sellers Ranking

Returns the top 5 sellers based on revenue for a specified date range.

**Endpoint**: `GET /functions/v1/transactions/ranking`

**Query Parameters**:
- `startDate` (required): Start date in ISO 8601 format (YYYY-MM-DD)
- `endDate` (required): End date in ISO 8601 format (YYYY-MM-DD)

**Response**: `200 OK`
```json
{
  "success": true,
  "period": {
    "startDate": "2025-01-01",
    "endDate": "2025-05-31"
  },
  "data": [
    {
      "position": 1,
      "sellerId": "company-uuid-1",
      "revenue": 150000.00,
      "sellerName": "Empresa ABC Ltda"
    },
    {
      "position": 2,
      "sellerId": "company-uuid-2",
      "revenue": 120000.00,
      "sellerName": "Comércio XYZ"
    }
    // ... up to position 5
  ]
}
```

**Error Responses**:
- `400 Bad Request`: Missing or invalid date parameters
- `403 Forbidden`: Access denied (non-admin users)
- `500 Internal Server Error`: Server error

**Example Request**:
```bash
curl -u "apikey:YOUR_COMPANY_ID" \
  "https://api.example.com/functions/v1/transactions/ranking?startDate=2025-01-01&endDate=2025-05-31"
```

---

### 2. Get Company Details

Returns detailed information about a specific company/seller.

**Endpoint**: `GET /functions/v1/transactions/ranking/{companyId}`

**Path Parameters**:
- `companyId` (required): The UUID of the company

**Response**: `200 OK`
```json
{
  "success": true,
  "id": "company-uuid",
  "name": "Empresa ABC Ltda",
  "cnpj": "12.345.678/0001-90",
  "responsavel": {
    "id": "user-uuid",
    "nome": "João Silva",
    "email": "joao@empresa.com",
    "telefone": "+5511999999999",
    "documento": "123.456.789-00",
    "dataNascimento": "1990-01-15",
    "criadoEm": "2023-06-10T10:30:00Z"
  }
}
```

**Error Responses**:
- `400 Bad Request`: Missing company ID
- `403 Forbidden`: Access denied (non-admin users)
- `404 Not Found`: Company not found
- `500 Internal Server Error`: Server error

**Example Request**:
```bash
curl -u "apikey:YOUR_COMPANY_ID" \
  "https://api.example.com/functions/v1/transactions/ranking/ranking/550e8400-e29b-41d4-a716-446655440000"
```

---

## Notes

1. **Access Control**: Both endpoints are restricted to admin users only
2. **Date Format**: All dates must be in ISO 8601 format (YYYY-MM-DD)
3. **Revenue Calculation**: Only transactions with status "paid" are included in the ranking
4. **Authentication**: The Basic Auth password should be your actual company ID, not the placeholder shown in examples
