# SimpleChat Endpoint Reference

Version: 0.230.001  
Generated: (update date as needed)  
Scope: Enumerates currently implemented HTTP routes (Flask `@app.route` and major Blueprints) for the `single_app` deployment.

> NOTE: This is an operational reference, not a formal OpenAPI spec. For an authoritative machine-readable contract, see `/swagger.json`.

---

## Table of Contents

1. Core & Static
2. Health & Swagger
3. Authentication Flow
4. Frontend Pages (User / Admin)
5. Chat & Conversations APIs
6. Documents (User / Group / Public / External)
7. Workflow & Bulk Operations
8. Prompts (User / Group / Public)
9. Groups & Public Workspaces (Mgmt APIs)
10. Agents & Orchestration
11. Models
12. OpenAPI Spec Management
13. Enhanced Citations & Media
14. Safety & Moderation
15. Feedback
16. User & Profile Utilities
17. Admin AI Search / Settings
18. Example & Misc
19. Feature Flags Summary
20. Load / Monitoring Recommendations
21. Programmatic Route Introspection
22. Pending / Not Fully Enumerated (Plugins)
23. Quick Cheat Sheet

---

## 1. Core & Static

| Path | Methods | Auth | Description | File |
|------|---------|------|-------------|------|
| `/` | GET | Public | Landing page (Markdown → HTML) | `app.py` |
| `/robots933456.txt` | GET | Public | Robots file (nonstandard name) | `app.py` |
| `/favicon.ico` | GET | Public | Favicon | `app.py` |
| `/acceptable_use_policy.html` | GET | Public | AUP page | `app.py` |
| `/static/js/<path:filename>` | GET | Public | JS assets (.mjs served with correct MIME) | `app.py` |

Example:
```bash
curl -i https://yourapp/
```

---

## 2. Health & Swagger

| Path | Methods | Flag | Auth | Description | File |
|------|---------|------|------|-------------|------|
| `/external/healthcheck` | GET | `enable_external_healthcheck` MUST BE SET TO `True` in Cosmos `settings` container | Public | Timestamp-based availability | `route_external_health.py` |
| `/swagger` | GET | — | Public | Swagger UI page | `swagger_wrapper.py` |
| `/swagger.json` | GET | — | Public | OpenAPI JSON spec | `swagger_wrapper.py` |
| `/api/swagger/routes` | GET | — | (Likely Auth) | Lists collected route metadata | `swagger_wrapper.py` |
| `/api/swagger/cache` | GET, DELETE | — | (Likely Auth) | Inspect / clear internal swagger cache | `swagger_wrapper.py` |

Healthcheck:
```bash
curl -i https://yourapp/external/healthcheck
```

If 400: enable `enable_external_healthcheck`.

---

## 3. Authentication Flow

| Path | Methods | Description | File |
|------|---------|-------------|------|
| `/login` | GET | Initiate Azure AD login | `route_frontend_authentication.py` |
| `/getAToken` | GET | Redirect URI (front-end flow) | same |
| `/getATokenApi` | GET | Token exchange (API variant) | same |
| `/logout` | GET | Session termination | same |

---

## 4. Frontend Pages (User / Admin Views)

| Path | Methods | Role | Description | File |
|------|---------|------|-------------|------|
| `/profile` | GET | User | Profile UI | `route_frontend_profile.py` |
| `/workspace` | GET | User | Workspace page | `route_frontend_workspace.py` |
| `/chats` | GET | User | Chat UI | `route_frontend_chats.py` |
| `/conversations` | GET | User | Conversation list UI | `route_frontend_conversations.py` |
| `/conversation/<conversation_id>` | GET | User | Conversation detail | `route_frontend_conversations.py` |
| `/conversation/<conversation_id>/messages` | GET | User | Messages view | same |
| `/group_workspaces` | GET | User | Group workspaces page | `route_frontend_group_workspaces.py` |
| `/my_groups` | GET | User | User’s groups | `route_frontend_groups.py` |
| `/my_public_workspaces` | GET | User | User’s public workspaces | `route_frontend_public_workspaces.py` |
| `/public_workspaces` | GET | User | Public workspaces list | same |
| `/public_directory` | GET | User | Public directory view | same |
| `/admin/settings` | GET/POST | Admin | Application settings UI | `route_frontend_admin_settings.py` |
| `/admin/safety_violations` | GET | Admin | Safety admin page | `route_frontend_safety.py` |
| `/safety_violations` | GET | User | User safety violation view | same |
| `/admin/feedback_review` | GET | Admin | Feedback moderation UI | `route_frontend_feedback.py` |
| `/my_feedback` | GET | User | User feedback list | same |

---

## 5. Chat & Conversations APIs

| Path | Methods | Auth | Description | File |
|------|---------|------|-------------|------|
| `/api/chat` | POST | Yes | Core chat interaction (model selection, search, orchestration) | `route_backend_chats.py` |
| `/api/get_messages` | GET | Yes | List messages (query params) | `route_backend_conversations.py` |
| `/api/get_conversations` | GET | Yes | List conversations | same |
| `/api/create_conversation` | POST | Yes | Create new conversation | same |
| `/api/conversations/<conversation_id>` | PUT/DELETE | Yes | Update (rename)/Delete | same |
| `/api/delete_multiple_conversations` | POST | Yes | Batch delete | same |
| `/api/conversations/<conversation_id>/metadata` | GET | Yes | Conversation metadata | same |
| `/api/message/<message_id>/metadata` | GET | Yes | Single message metadata | `route_frontend_conversations.py` |
| `/api/image/<image_id>` | GET | Yes | Retrieve generated image | `route_backend_conversations.py` |

Chat request example:
```bash
curl -X POST https://yourapp/api/chat \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
        "conversation_id": "conv-123",
        "message": "Hello!",
        "model_deployment": "gpt-4o",
        "hybrid_search": false
      }'
```

---

## 6. Documents (User / Group / Public / External)

### User Documents

| Path | Methods | Description | File |
|------|---------|-------------|------|
| `/api/documents/upload` | POST | Upload (multipart) | `route_backend_documents.py` |
| `/api/documents` | GET | List documents | same |
| `/api/documents/<document_id>` | GET/PATCH/DELETE | CRUD | same |
| `/api/documents/<document_id>/extract_metadata` | POST | Trigger metadata extraction | same |
| `/api/documents/upgrade_legacy` | POST | Upgrade legacy format | same |
| `/api/documents/<document_id>/share` | POST | Share with users | same |
| `/api/documents/<document_id>/unshare` | DELETE | Revoke sharing | same |
| `/api/documents/<document_id>/shared-users` | GET | List shared users | same |
| `/api/documents/<document_id>/approve-share` | POST | Approve share request | same |
| `/api/documents/<document_id>/remove-self` | DELETE | Remove own access | same |
| `/api/get_file_content` | POST | Retrieve filtered content | same |
| `/api/get_citation` | POST | Generate citation | same |

Upload example:
```bash
curl -X POST https://yourapp/api/documents/upload \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@/path/to/file.pdf"
```

### Group Documents (Parallel Semantics)

Prefix: `/api/group_documents` (upload, list, CRUD, extract, upgrade, sharing with groups).  
File: `route_backend_group_documents.py`.

### Public Documents (Internal Auth)

Prefix: `/api/public_documents` and `/api/public_workspace_documents` (list, CRUD, extraction, upgrade).  
File: `route_backend_public_documents.py`.

### External Public Document API

Prefix: `/external/public_documents` (mirrors upload/list/CRUD/extract/upgrade).  
File: `route_external_public_documents.py`.

---

## 7. Workflow & Bulk Operations

| Path | Methods | Type | Description | File |
|------|---------|------|-------------|------|
| `/workflow/*` | GET/POST (varied) | UI | Multi-step bulk / summary flows | `route_frontend_workflow.py` |
| `/api/workflow/generate-summary` | POST | API | Create summary | same |
| `/api/workflow/generate-pii-analysis` | POST | API | PII analysis | same |
| `/api/workflow/bulk-process` | POST | API | Start bulk job | same |
| `/api/workflow/bulk-status/<job_id>` | GET | API | Check bulk status | same |

---

## 8. Prompts (User / Group / Public)

| Scope | Paths | File |
|-------|-------|------|
| User | `/api/prompts` (GET/POST), `/api/prompts/<id>` (GET/PATCH/DELETE) | `route_backend_prompts.py` |
| Group | `/api/group_prompts` (GET/POST), `/api/group_prompts/<id>` (GET/PATCH/DELETE) | `route_backend_group_prompts.py` |
| Public | `/api/public_prompts` (GET/POST), `/api/public_prompts/<id>` (GET/PATCH/DELETE) | `route_backend_public_prompts.py` |

---

## 9. Groups & Public Workspaces

### Groups

Representative endpoints (all in `route_backend_groups.py`):

- Discover/list/create/delete/update:
  - `/api/groups` (GET/POST)
  - `/api/groups/<group_id>` (GET/PATCH/PUT/DELETE)
- Membership & requests:
  - `/api/groups/<group_id>/members` (GET/POST)
  - `/api/groups/<group_id>/members/<member_id>` (DELETE/PATCH)
  - `/api/groups/<group_id>/requests` (GET/POST)
  - `/api/groups/<group_id>/requests/<request_id>` (PATCH)
- Activity / counts / ownership:
  - `/api/groups/setActive` (PATCH)
  - `/api/groups/<group_id>/fileCount` (GET)
  - `/api/groups/<group_id>/transferOwnership` (PATCH)

### Public Workspaces

File: `route_backend_public_workspaces.py`

- `/api/public_workspaces` (GET/POST)
- `/api/public_workspaces/<ws_id>` (GET/PATCH/PUT/DELETE)
- Requests & membership (`/requests`, `/members`)
- Ownership transfer, file/prompt counts:
  - `/api/public_workspaces/<ws_id>/fileCount`
  - `/api/public_workspaces/<ws_id>/promptCount`

Frontend activation helpers:
- `/set_active_group` (POST)
- `/set_active_public_workspace` (POST)

---

## 10. Agents & Orchestration (Blueprint)

File: `route_backend_agents.py` (Blueprint)

| Path | Methods | Flag (if any) | Notes |
|------|---------|---------------|-------|
| `/api/agents/generate_id` | GET | — | New GUID |
| `/api/user/agents` | GET/POST | POST requires `allow_user_agents` | List/update personal agents |
| `/api/user/agents/<agent_name>` | DELETE | `allow_user_agents` | Delete personal agent |
| `/api/user/settings/selected_agent` | POST | — | Set active agent |
| `/api/user/agent/settings` | GET | — | User agent settings view |
| `/api/admin/agent/settings` | GET | — | Admin view |
| `/api/admin/agents` | GET/POST | — | Global agents list/create |
| `/api/admin/agents/<agent_name>` | PUT/DELETE | — | Modify/delete global agent |
| `/api/admin/agents/settings/<setting_name>` | GET/POST | — | Specific agent settings |
| `/api/orchestration_types` | GET | — | Available orchestration modes |
| `/api/orchestration_settings` | GET/POST | — | Orchestration config |

---

## 11. Models

File: `route_backend_models.py`

| Path | Methods | Description |
|------|---------|-------------|
| `/api/models/gpt` | GET | GPT model deployments |
| `/api/models/embedding` | GET | Embedding models |
| `/api/models/image` | GET | Image generation models |

---

## 12. OpenAPI Spec Management

File: `route_openapi.py`

| Path | Methods | Description |
|------|---------|-------------|
| `/api/openapi/upload` | POST | Upload spec file |
| `/api/openapi/validate-url` | POST | Validate remote spec |
| `/api/openapi/download-from-url` | POST | Fetch remote spec |
| `/api/openapi/list-uploaded` | GET | List stored specs |
| `/api/openapi/analyze-auth` | POST | Inspect auth schemes |

---

## 13. Enhanced Citations & Media

File: `route_enhanced_citations.py`

| Path | Methods | Media |
|------|---------|-------|
| `/api/workflow/pdf` | GET | Workflow PDF |
| `/api/enhanced_citations/image` | GET | Image artifact |
| `/api/enhanced_citations/video` | GET | Video artifact |
| `/api/enhanced_citations/audio` | GET | Audio artifact |
| `/api/enhanced_citations/pdf` | GET | Enhanced PDF |

---

## 14. Safety & Moderation

File: `route_backend_safety.py`

| Path | Methods | Audience | Description |
|------|---------|----------|-------------|
| `/api/safety/logs` | GET | Admin | All logs |
| `/api/safety/logs/<log_id>` | PATCH | Admin | Update status |
| `/api/safety/logs/my` | GET | User | Own logs |
| `/api/safety/logs/my/<log_id>` | PATCH | User | Update own entry |

---

## 15. Feedback

File: `route_backend_feedback.py`

| Path | Methods | Role | Description |
|------|---------|------|-------------|
| `/feedback/submit` | POST | User | Submit feedback |
| `/feedback/review` | GET | Admin | List |
| `/feedback/review/<feedbackId>` | GET/PATCH | Admin | View/update |
| `/feedback/retest/<feedbackId>` | POST | Admin | Retest scenario |
| `/feedback/my` | GET | User | User feedback list |

---

## 16. User & Profile Utilities

Files: `route_backend_users.py`, `route_frontend_profile.py`

| Path | Methods | Description |
|------|---------|-------------|
| `/api/userSearch` | GET | User search |
| `/api/user/info/<user_id>` | GET | Basic user info |
| `/api/user/settings` | GET/POST | Read/update user settings |
| `/api/user/profile-image/<user_id>` | GET | Profile image |
| `/api/profile/image/refresh` | POST | Refresh cached profile image |

---

## 17. Admin AI Search / Settings

File: `route_backend_settings.py`

| Path | Methods | Description |
|------|---------|-------------|
| `/api/admin/settings/check_index_fields` | POST | Validate existing index |
| `/api/admin/settings/fix_index_fields` | POST | Patch index fields |
| `/api/admin/settings/create_index` | POST | Create index |
| `/api/admin/settings/test_connection` | POST | Test AI Search connectivity |

---

## 18. Example & Misc

| Path | Methods | Description | File |
|------|---------|-------------|------|
| `/api/example` | POST | Sample swagger-wrapped endpoint | `swagger_wrapper.py` |
| `/upload` | POST | Legacy/Chat file upload | `route_frontend_chats.py` |
| `/view_pdf` | GET | View PDF (legacy) | `route_frontend_chats.py` |
| `/view_document` | GET | View generic document | same |
| `/set_active_group` | POST | Set active group | `route_frontend_group_workspaces.py` |
| `/set_active_public_workspace` | POST | Set active public workspace | `route_frontend_public_workspaces.py` |

---

## 19. Feature Flags (Selected)

Flag keys from `functions_settings.get_settings()` impacting routing:

| Flag | Affects |
|------|---------|
| `enable_external_healthcheck` | `/external/healthcheck` |
| `allow_user_agents` | POST/DELETE on user agents |
| `enable_content_safety` | Safety handling in chat (not a route guard) |
| `enable_gpt_apim` / `enable_embedding_apim` / `enable_image_gen_apim` | Model resolution behavior |
| `enable_enhanced_citations` | Enhanced citation retrieval endpoints |
| `enable_user_workspace`, `enable_group_workspaces`, `enable_public_workspaces` | Visibility & logic (routes remain but may error) |
| `enable_user_feedback` | Feedback submission |
| `enable_file_sharing` | Document sharing flows |

Routes decorated with `@enabled_required("flag")` return `400` with JSON if disabled.

---

## 20. Load / Monitoring Recommendations

| Goal | Suggested Endpoint |
|------|--------------------|
| Liveness / simple uptime | `/external/healthcheck` |
| Full landing render | `/` |
| Auth check latency | `/api/models/gpt` |
| Heavy business logic | `/api/chat` |
| Storage & I/O | `/api/documents/upload` |
| Bulk workflow | `/api/workflow/bulk-process` |

Warm up the app (several serial hits) before high-concurrency benchmarking to avoid cold-start skew.

---

## 21. Programmatic Route Introspection

Optional snippet (protect behind admin/auth):

```python
for rule in app.url_map.iter_rules():
    methods = ",".join(sorted(m for m in rule.methods if m not in ['HEAD','OPTIONS']))
    print(f"{rule.rule:60s} {methods:20s} -> {rule.endpoint}")
```

---

## 22. Pending / Not Fully Enumerated (Plugins)

`route_backend_plugins.py` uses a Blueprint (not captured by a simple `@app.route` grep). If you need plugin CRUD endpoints:
- Search for `.route('/api/` within that file.
- Endpoints dynamically manage Semantic Kernel plugins (validation, metadata, health checks).

---

## 23. Quick Cheat Sheet (Common)

| Purpose | Path |
|---------|------|
| Health | `/external/healthcheck` |
| Swagger | `/swagger.json` |
| Chat | `/api/chat` |
| Conversations | `/api/get_conversations` |
| Documents | `/api/documents` |
| Upload | `/api/documents/upload` |
| Models | `/api/models/gpt` |
| Agents | `/api/user/agents` |
| Public Workspaces | `/api/public_workspaces` |

---

## Change Management

If you add new routes:
1. Update this file.
2. Regenerate `/swagger.json` (auto on app restart).
3. Optionally add functional tests under `functional_tests/`.

---

## Disclaimer

- Some routes may still perform settings lookups or Cosmos DB reads even if they appear "lightweight".
- Authentication: Many “public” routes still may rely on session cookies for full UI rendering; protect sensitive ones behind decorators.
- This document may drift—use automated extraction to keep updated.