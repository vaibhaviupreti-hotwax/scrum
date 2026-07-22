Searched for "checkLoginOptions"
Viewed localApiServerDiscovery.ts:40-65

In the Moqui Framework and this codebase, the login REST API endpoints are configured as follows:

---

### Moqui REST Login Endpoints (from Codebase Trace)

1. **Primary Moqui Login Endpoint** (see [useAuth.ts:L85](file:///home/vaibhaviupreti/1-HW/freq-used/13-accxui/common/composables/useAuth.ts#L85)):
   * **URL**: `POST http://localhost:8080/rest/s1/admin/login`
   * **Method**: `POST` *(Must be `POST`, `GET` returns `404`)*
   * **Headers**: `Content-Type: application/json`
   * **Body (`raw` -> `JSON`)**:
     ```json
     {
       "username": "<your_username>",
       "password": "<your_password>"
     }
     ```

2. **Moqui Check Login Options** (see [localApiServerDiscovery.ts:L52](file:///home/vaibhaviupreti/1-HW/freq-used/13-accxui/common/core/localApiServerDiscovery.ts#L52)):
   * **URL**: `GET http://localhost:8080/rest/s1/admin/checkLoginOptions`
   * **Method**: `GET`
   * *(Used to probe if the Moqui REST server is running and check authentication options)*

3. **Alternative Moqui Root REST Route**:
   * **URL**: `POST http://localhost:8080/rest/s1/login`
   * **Method**: `POST`
   * **Body**:
     ```json
     {
       "username": "<your_username>",
       "password": "<your_password>"
     }
     ```

4. **OFBiz REST Login Endpoint** *(if running OFBiz instead of Moqui)*:
   * **URL**: `POST http://localhost:8080/api/login`
   * **Method**: `POST`
   * **Body**:
     ```json
     {
       "USERNAME": "<your_username>",
       "PASSWORD": "<your_password>"
     }
     ```

---

### Summary for your Postman Test:
To login on local Moqui:
Set **Method** = `POST`, **URL** = `http://localhost:8080/rest/s1/admin/login`, **Body** = `{"username": "...", "password": "..."}`.
