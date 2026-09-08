# Reverse Engineering APIs

---

Two methods for discovering and documenting all of crAPI’s endpoints by capturing real traffic, instead of guessing at routes.

## Method 1 — Using Postman

1. Create a new **Workspace** → create a **Collection**.
2. Bottom-right corner → **Tools** → **Proxy** → enable **Proxy** (pick a free port, e.g. `5555`) → click **Start Capture**.
    
    ![Screenshot From 2026-09-08 02-54-06.png](Reverse%20Engineering%20APIs/Screenshot_From_2026-09-08_02-54-06.png)
    
    ![Screenshot From 2026-09-08 02-54-45.png](Reverse%20Engineering%20APIs/Screenshot_From_2026-09-08_02-54-45.png)
    
3. On the machine running crAPI, open **Firefox** with the **FoxyProxy** extension → point it at the same port (`8082` or `5555` — double-check which one is actually active/listening).
4. With the proxy capturing, exercise every feature of the app: sign up, sign in, place an order, request a refund, etc. — hit as many flows/features as possible so every endpoint gets captured.
5. Once done, go back to Postman → select **all** captured requests in the workspace/collection view → **Add all to collection**.
6. Create a folder named `127.0.0.1` (or whatever the host is).
7. Sort the captured links by API namespace into subfolders, e.g.: `community`, `identity`, `workshop`.
8. Inside `identity`, create an additional subfolder for **v2 API** routes, and move the relevant v2 endpoints into it.

![Screenshot From 2026-09-08 01-55-53.png](Reverse%20Engineering%20APIs/Screenshot_From_2026-09-08_01-55-53.png)

Result: a fully organized Postman collection mirroring crAPI’s real API surface, sorted by service/version.

## Method 2 — Using mitmweb + mitmproxy2swagger

1. Set up **FoxyProxy** to listen on the same port as mitmweb (same setup as Method 1).

![Screenshot From 2026-09-07 23-05-00.png](Reverse%20Engineering%20APIs/Screenshot_From_2026-09-07_23-05-00.png)

1. Exercise every endpoint/feature in the app again (sign up, sign in, orders, refunds, video upload, etc.) while mitmweb captures traffic.
2. Once done: in mitmweb, go to **File** → **Save as** → save the flows file.
3. Stop the interception.
4. Go to the **Downloads** folder — the `flows` file should be there.
5. Use **Swagger** (via `mitmproxy2swagger`) to turn the captured flows into an OpenAPI spec.

### Step-by-step: generating the Swagger spec

**Step 1 — Activate your Python environment**

Open a terminal, go to your Downloads folder, and activate the virtual environment you created earlier:

```bash
cd ~/Downloads
source ~/mitm_env/bin/activate
```

**Step 2 — First run (build the initial route list)**

Generate a first-pass list of API routes from the flows file (no `sudo`, no extra flags):

```bash
mitmproxy2swagger -i flows -o spec.yml -p http://127.0.0.1:8888 -f flow
```

![Screenshot From 2026-09-08 03-51-12.png](Reverse%20Engineering%20APIs/Screenshot_From_2026-09-08_03-51-12.png)

**Step 3 — Sed command (unlock all endpoints)**

The initial spec marks many routes with `ignore:` tags. Strip all of them in one shot so the tool will process every route:

```bash
sed -i 's/ignore://g' spec.yml
```

**Step 4 — Second run (build the full schema with real data)**

Re-run the same command to pull the actual data (HTTP method, headers, body) for the now-unlocked routes back from the flows file, producing a complete, working Swagger file:

```bash
mitmproxy2swagger -i flows -o spec.yml -p http://127.0.0.1:8888 -f flow
```

**Step 5 — Import and clean up in Swagger Editor**

1. Go to [editor.swagger.io](https://editor.swagger.io/).
2. **Import File** → load `spec.yml`.
3. All endpoints should now be visible as a well-documented API spec.
4. Use the **Edit** option on the left panel to rename the title to `crAPI`.
5. Download the finished spec.
    
    ![Screenshot From 2026-09-08 04-35-56.png](Reverse%20Engineering%20APIs/Screenshot_From_2026-09-08_04-35-56.png)
    

**Step 6 — Bring it into Postman**

Import the downloaded Swagger/OpenAPI document into the **same Postman collection** created in Method 1 — it now sits alongside the manually sorted folders from the first method, giving you two cross-checked views of the same API surface.