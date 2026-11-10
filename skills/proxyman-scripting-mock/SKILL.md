---
name: proxyman-scripting-mock
description: Backend mocks via Proxyman Scripting — create, update, disable a task mock script. Use on "mock endpoint", "mock in proxyman", "backend is down", "fix mock", "what is mocked".
license: MIT
compatibility: Requires Proxyman with MCP enabled and the Proxyman MCP server connected.
metadata:
    type: task
    status: test
---

| Name | Description |
|---|---|
| Proxyman Scripting Mock | Backend mocks as Proxyman Scripting rules, mirrored from `.js` files on disk. |

# Proxyman Scripting Mock

Source of truth: file on disk. Proxyman mirrors it.

## Layout

| | Path |
|---|---|
| Disk | `~/Proxyman/Scripting/[<Company>/]<Product>/[Testing/]<name>.js` |
| Proxyman | Scripting rule folder `[<Company>/]<Product>[/Testing]`, rule `<name>` |

- Disk root splits by Proxyman tool: `Scripting`, `Map Local`. This skill: `Scripting`.
- `<Company>`: only if project belongs to company (e.g. `Acme`). Else omit level.
- `<Product>`: product or app name, e.g. `Dashboard`, `Admin`.
- Purpose: ask user — task mock or global testing mock. Testing → `Testing` folder inside product.
- `<name>`: `[<Env>[-<TaskID>]] <Entity> [<Version>] — <short description>`. File name = rule name.
    - `<Env>`: contour of mocked host — `Dev`, `Stage`, etc.
    - `-<TaskID>`: task mocks only. Testing mocks: `[<Env>]`.
    - `<Version>`: only if entity has versions.
    - Examples: `[Dev-123] Orders — 6 orders in all states`, `[Dev-123] User settings V2 — PUT saves, GET returns saved`, `[Stage] Feature flags — all enabled`.
- One script per task. All task endpoints in one `route()`.

## File

Top to bottom:

1. Header: one block comment `/* … */`, no line prefixes.

    ```js
    /*
    Mock reason: <why>
    Task: <task link>
    Repository: <GitLab or GitHub branch / MR / PR link>

    Endpoints:
    L<line> [METHOD-STATUS] <full path>
    */
    ```

    `Endpoints`: table of contents, one line per endpoint block, `<line>` = line where its `// [METHOD-STATUS]` comment starts. Any edit shifting lines → update.

2. `// Shared helpers`: shared state (`sharedState`), small helpers used by several endpoints. Right after header.
3. One block per endpoint, in this order:
    - `// [METHOD-STATUS] <full path>` — `METHOD`: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, etc. `STATUS`: mocked response code, default `200`; user may request other (`[PUT-202]`, `[GET-404]`).
    - `// <what mock does, short>`
    - Response JSON: `UPPER_SNAKE_CASE` constant, pretty-printed (one key per line, 4-space indent), any size. No external files, no `bodyFilePath`.
    - Endpoint handler and its modifications.

    ```js
    // [GET-200] /users/{userId}/orders
    // Six orders in all states
    const USER_ORDERS = [
        {
            "orderId": "6ab461b0-6451-44c4-bf3f-c37f736527e0",
            "title": "Order 1",
            "state": "pending"
        }
    ];

    const getUserOrders = () => USER_ORDERS;
    ```

4. `route(method, path, request)`: dispatches to endpoint handlers; unknown → `undefined`.
5. Proxyman handlers: canonical arguments `(context, url, request, response)`, arrow form `const onResponse = async (context, url, request, response) => { … };`. Read `docs://scripting/snippet-code` (proxyman MCP) first.

Mode:

- Mock API (`is_mock_response: true`): backend down. No upstream. Answer `OPTIONS` preflight: `204`, `Access-Control-Allow-Origin` = request `Origin`, `Allow-Credentials`, `Allow-Methods`, `Allow-Headers` = `Access-Control-Request-Headers`. Unknown endpoint: `503` + `X-Mocked-By`.
- Response override (`is_mock_response: false`): backend up. Change task endpoints only, pass rest.

Cross-request state (PUT → GET): `sharedState`.

Text: all text in script and rule — header, comments, names, mock reason — short, telegraphic, English. No intro words, no full sentences where a phrase works.

Code style: arrow functions only (`const name = (…) => …`), no `function` declarations. Functional array methods (`find`, `map`, `filter`, `some`), no `for`/`for…in`/`for…of`/`while`.

## Steps

1. Collect: purpose (task or testing), env, task ID, task link, repository link, reason, company, product, entity and version, endpoints (method, path, status, response), mode. Response from API types or real flow (`get_flow_detail`). Missing → ask.
2. `list_rules`: rule on same URLs exists → show user, decide update or disable.
3. Write `.js` per File.
4. `list_rule_folders` (`rule_type: scripting`). Folder missing → `create_rule_folder`.
5. `create_scripting_rule`: `name` = file name, `folder` = disk folder path, `url` = regex on host + task paths, `script_content` = file content, mode from step 1.
6. Check each endpoint: `curl -k -x http://127.0.0.1:9090 -H "Origin: <app origin>" <url>` → status, body, `X-Mocked-By`. Mock API → check `OPTIONS` too.

Done: file and rule match in name, folder, content; every endpoint from step 1 returns mock.

## Update, disable

- Update: file first, then `update_scripting_rule` same content. File ≠ rule → error.
- Disable: `toggle_rule`. Keep file and rule. Delete only on direct request.
- What is mocked: `list_rules` + files in `~/Proxyman/`.
