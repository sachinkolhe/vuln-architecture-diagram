# Fast Search Vulnerability For Customer
Here’s a clean, structured, and professional reformatting of your content with clearer wording, consistent terminology, and better flow—without changing the intent or technical details.

---

## Requirement

We need an API that retrieves data from the database based on search queries provided by the customer.

**Example search conditions:**

* vuln.cvssScore > `5.0` and vuln.severity : `HIGH`

---

## Non-Functional Requirements

* The search service must be **highly available**.
* The service must provide **low-latency** search responses.

---

## System Components

<img width="458" height="198" alt="image" src="https://github.com/user-attachments/assets/b955f7e0-1d29-4996-a6a8-e881754bd8f3" />

The system consists of the following three components:

1. **UI Service**
   Serves the web application and provides the search interface to users.

2. **Backend Service**
   Accepts search requests from the UI and processes them.

3. **Datastores**

   * **Elasticsearch** for fast search and filtering
   * **Oracle Database** as the system of record

---

## API Specification

### Endpoint

```
GET /search
```

### Query Parameters

```
query = "vuln.cvssScore > 5.0 AND vuln.severity: HIGH"
pageNumber = 0
pageSize = 100
```

### Response

Returns a list of assets that match the vulnerability criteria.

```json
[
  {
    "asset_id": "asset-uuid-10",
    "asset_name": "prod-db-01",
    "asset_type": "VM",
    "ip_address": "10.10.1.5",
    "vulnerability_id": "vuln-uuid-1",
    "cve_id": "CVE-2024-1234",
    "title": "Remote Code Execution",
    "severity": "CRITICAL",
    "cvss_score": 9.8
  }
]
```

---

## Elasticsearch Index Mapping

```json
PUT asset_vulnerabilities_search
{
  "mappings": {
    "properties": {
      "customer_id": {
        "type": "keyword"
      },

      "asset": {
        "properties": {
          "asset_id": { "type": "keyword" },
          "asset_name": {
            "type": "text",
            "fields": {
              "keyword": { "type": "keyword" }
            }
          },
          "asset_type": { "type": "keyword" },
          "ip_address": { "type": "ip" }
        }
      },

      "vulnerabilities": {
        "type": "nested",
        "properties": {
          "vulnerability_id": { "type": "keyword" },
          "cve_id": { "type": "keyword" },
          "title": { "type": "text" },
          "severity": { "type": "keyword" },
          "cvss_score": { "type": "float" },
          "status": { "type": "keyword" },
          "source": { "type": "keyword" }
        }
      },
      "first_seen": { "type": "date" },
      "last_seen": { "type": "date" }
    }
  }
}
```

---

## Oracle Database Tables

* `customers`
* `assets`
* `vulnerabilities`
* `asset_vulnerabilities`


<img width="638" height="182" alt="image" src="https://github.com/user-attachments/assets/b09f0b38-93c0-4789-b297-5de67a55b75e" />

---

## Search Flow

1. The user enters a search query in the UI search box.
2. The backend parses the query using an **ANTLR-based grammar**.
3. The parsed query is converted into an **Elasticsearch query**.
4. The Elasticsearch query is executed against the search index.
5. Elasticsearch returns document IDs, which correspond to the **primary key of the `asset_vulnerabilities` table** in Oracle.
6. The backend fetches the complete data from Oracle using this ID.

   * The ID is defined as a primary key and is indexed for efficient lookup.


