# Test Design Techniques

## Boundary Value Analysis (BVA)

Test at the edges of valid ranges, not the middle.

| Type | Values to test |
|---|---|
| Integer | `0`, `-1`, `min`, `min-1`, `max`, `max+1`, `2147483647`, `-2147483648` |
| String length | `""`, `1 char`, `max length`, `max+1 chars`, 256 chars (if no max documented) |
| Array | `[]`, single element, 100 elements |
| Date | today, past date, future date, epoch `0`, year `9999` |

## Equivalence Partitioning (EP)

Group inputs into classes where all values behave the same, test one from each.

- **Valid class**: typical correct value
- **Invalid class**: wrong type, out of range, malformed format
- **Edge class**: boundary values above

For email: `valid@example.com` (valid), `notanemail` (invalid), `a@b` (edge).
For UUID: valid v4, all-zeros `00000000-0000-0000-0000-000000000000`, non-UUID string.

## Error Guessing

Payloads that commonly expose bugs:

**SQL injection**
```
' OR '1'='1
'; DROP TABLE users;--
1 OR 1=1
```

**XSS**
```
<script>alert(1)</script>
"><img src=x onerror=alert(1)>
javascript:alert(1)
```

**Path traversal**
```
../../../etc/passwd
..%2F..%2Fetc%2Fpasswd
....//....//etc/passwd
```

**Format strings / special chars**
```
%s%s%s%s
{{7*7}}
${7*7}
\x00null\x00byte
```

**Size**
- 10,000-character string
- Integer `999999999999`
- Deeply nested JSON `{"a":{"a":{"a": ...}}}` (20 levels)

## State Machine

For CRUD endpoint groups, test the full lifecycle:

1. POST → capture created ID → assert 201
2. GET `/{id}` → assert fields match what was posted
3. PUT/PATCH `/{id}` → assert 200
4. GET `/{id}` → assert updated fields
5. DELETE `/{id}` → assert 204 or 200
6. GET `/{id}` → assert 404

Also test out-of-order operations: DELETE before CREATE, double DELETE, PATCH after DELETE.

## IDOR (Insecure Direct Object Reference)

Requires knowing a resource ID belonging to a *different* user.

Sources for IDs:
1. `session-memory.json` → `known_ids` (populated automatically by explorer)
2. Sequential guess: if your resource is ID 123, try 122 and 124
3. Manually provided by the user via `/session add-id`

For each parameterized endpoint (e.g. `GET /users/{id}`):
- Authenticate as user A
- Request a resource owned by user B
- Expected: 403 Forbidden or 404 Not Found
- Issue if: 200 with user B's data returned

## Mass Assignment

Include fields not in the documented request schema:

```json
{
  "name": "John",
  "email": "john@example.com",
  "role": "admin",
  "is_admin": true,
  "is_verified": true,
  "balance": 999999,
  "permissions": ["read", "write", "admin"]
}
```

A 2xx response doesn't confirm acceptance — follow up with a GET to check if the injected fields persisted.
