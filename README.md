# **Zeotap – Salesforce Service Cloud Integration (Contact Object)**

**Author:** Aarush Dhawan  
**Role Applied For:** PM Integrations Intern  
**Date:** 12 April 2025 

## **1. Use Case and Goals**

The objective of this integration is to ensure that customer contact information in Salesforce Service Cloud remains accurate and up to date by syncing profile data from Zeotap. This enables customer service agents to access the most current user data, leading to enhanced customer experiences, more accurate personalization, and reduced operational friction.

### **Business Value:**
- Improved data consistency across platforms
- Streamlined customer service workflows
- Reduced manual data entry and errors
- Enhanced segmentation for marketing and support

## **2. Data Flow Overview**

### **Data Sync Scenario:**
1. A user updates their contact information in Zeotap (e.g., email, phone number, address).
2. Zeotap triggers a POST or PATCH request to Salesforce’s REST API.
3. Salesforce either:
   - Creates a new Contact if none exists.
   - Updates the existing Contact if a match is found (based on email or phone).
4. Salesforce returns a response indicating success or failure.

### **Flow Summary:**
- **Source**: Zeotap (data origin)
- **Destination**: Salesforce Service Cloud (Contact object)
- **Trigger**: User profile update in Zeotap

## **3. Authentication Method**

Salesforce uses **OAuth 2.0** for REST API authentication.

### **Steps:**
1. **Authorization Code Request**  
   `https://login.salesforce.com/services/oauth2/authorize`  
   Parameters:  
   - `response_type=code`  
   - `client_id=<CLIENT_ID>`  
   - `redirect_uri=<REDIRECT_URI>`

2. **Access Token Request**  
   `https://login.salesforce.com/services/oauth2/token`  
   Method: `POST`  
   Form Data:
   - `grant_type=authorization_code`
   - `code=<AUTHORIZATION_CODE>`
   - `client_id=<CLIENT_ID>`
   - `client_secret=<CLIENT_SECRET>`
   - `redirect_uri=<REDIRECT_URI>`

3. **API Request with Token**  
   Add `Authorization: Bearer <access_token>` to all REST API calls.

## **4. Key Salesforce REST API Endpoints**

| Action           | HTTP Method | Endpoint |
|------------------|-------------|----------|
| Create Contact   | POST        | `/services/data/v59.0/sobjects/Contact` |
| Update Contact   | PATCH       | `/services/data/v59.0/sobjects/Contact/{ContactId}` |
| Query Contact ID | GET         | `/services/data/v59.0/query/?q=SELECT+Id+FROM+Contact+WHERE+Email='user@example.com'` |

## **5. Sample Requests and Responses**

### **Create Contact Request**
```http
POST /services/data/v59.0/sobjects/Contact
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "FirstName": "Ram",
  "LastName": "Mehra",
  "Email": "ram.mehra@example.com",
  "Phone": "12345XXXXX"
}
```

**Response**
```json
{
  "id": "0035g00000XXXXXXAAO",
  "success": true,
  "errors": []
}
```

### **Update Contact Request**
```http
PATCH /services/data/v59.0/sobjects/Contact/0035g00000XXXXXXAAO
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "Phone": "98765XXXXX"
}
```

**Response:** HTTP 204 No Content

### **Query Contact ID by Email**
```http
GET /services/data/v59.0/query?q=SELECT+Id+FROM+Contact+WHERE+Email='john.doe@example.com'
Authorization: Bearer <access_token>
```

**Response**
```json
{
  "totalSize": 1,
  "records": [
    {
      "Id": "0035g00000XXXXXXAAO"
    }
  ]
}
```

## **6. Common Errors, Rate Limits, Required Fields**

### **Required Fields:**
- `LastName` is mandatory when creating a Contact.

### **Common Errors:**
| Error | Description |
|-------|-------------|
| `INVALID_SESSION_ID` | Token expired or invalid. |
| `DUPLICATE_VALUE` | Duplicate unique field like email or phone. |
| `REQUIRED_FIELD_MISSING` | Missing `LastName` or other required fields. |
| `INSUFFICIENT_ACCESS` | User does not have permission. |

### **Rate Limits:**
- **15,000 API calls/day** for standard Salesforce editions.
- Up to **100 concurrent REST API requests**.
- Handle limits with exponential backoff or retries after `Retry-After` header.

## **7. Assumptions and Edge Cases**

### **Assumptions:**
- Matching of existing Contacts is done using **email** (unique identifier).
- Zeotap will store Salesforce Contact ID once created to enable efficient updates.

### **Edge Cases:**
- **Missing email**: Cannot determine uniqueness; may cause duplicate records.
- **Multiple records with same email**: Ambiguous match; may require human intervention.
- **Network/API failures**: Retry with backoff logic.
- **Token expiration**: Refresh token should be implemented.

## **8. Testing Approach (Bonus)**

### **Unit Tests**
- Validate data mapping from Zeotap format to Salesforce Contact format.

### **Integration Tests**
- Use Salesforce Sandbox and mock Zeotap data updates.
- Test both Contact creation and update flows.

### **End-to-End Testing**
- Simulate real user updates in Zeotap.
- Monitor resulting Contact record in Salesforce.

### **Error Simulation**
- Test for expired tokens, rate limit breaches, duplicate data, and missing fields.

## **References**
- Salesforce REST API Documentation: [https://developer.salesforce.com/docs/apis](https://developer.salesforce.com/docs/apis)
