<!-- eva-primed-plan -->

## EVA Ecosystem Tools

- Data model: GET http://localhost:8010/model/projects/28-rbac
- 29-foundry agents: C:\eva-foundry\eva-foundation\29-foundry\agents\
- 48-eva-veritas audit: run audit_repo MCP tool

---

# Project Plan

<!-- veritas-normalized 2026-02-25 prefix=F28 source=README.md -->

## Feature: ?? Documentation Suite [ID=F28-01]

## Feature: Executive Summary [ID=F28-02]

## Feature: Table of Contents [ID=F28-03]

## Feature: Architecture Overview [ID=F28-04]

### Story: 3-Layer RBAC Model [ID=F28-04-001]

## Feature: Authorization Flow [ID=F28-05]

### Story: Visual Flow Diagram [ID=F28-05-001]

### Story: by-Step Process [ID=F28-05-002]

### Story: Permission Matrix by Role [ID=F28-05-003]

## Feature: Group Structure [ID=F28-06]

### Story: Naming Convention (CRITICAL) [ID=F28-06-001]

## Feature: Cosmos DB Schema [ID=F28-07]

### Story: Complete Database Architecture [ID=F28-07-001]

### Story: Database 1: Group-to-Resource Mapping [ID=F28-07-002]

### Story: Item Schema [ID=F28-07-003]

### Story: Field Descriptions [ID=F28-07-004]

### Story: Database 2: User Profile Management [ID=F28-07-005]

### Story: Database 3: Document Status Tracking [ID=F28-07-006]

### Story: Database 4: Custom Example Prompts [ID=F28-07-007]

### Story: Database 5: Conversation History [ID=F28-07-008]

### Story: Field Descriptions [ID=F28-07-009]

### Story: Azure RBAC Roles Reference [ID=F28-07-010]

## Feature: Data Flow Diagrams [ID=F28-08]

### Story: 1. User Login & Group Selection Flow [ID=F28-08-001]

### Story: 2. Document Upload & Processing Flow [ID=F28-08-002]

### Story: 3. Chat Request with RAG Flow [ID=F28-08-003]

## Feature: Implementation Details [ID=F28-09]

### Story: Core Files [ID=F28-09-001]

### Story: Code Examples [ID=F28-09-002]

### Story: API Endpoint Integration Pattern [ID=F28-09-003]

## Feature: Multi-Environment Setup [ID=F28-10]

### Story: Environment Architecture Diagram [ID=F28-10-001]

### Story: Environment Configurations [ID=F28-10-002]

### Story: Environment Switching [ID=F28-10-003]

## Feature: Debugging & Troubleshooting [ID=F28-11]

### Story: Common Issues [ID=F28-11-001]

- [ ] Issue 1: "You are not assigned to any project" [ID=F28-11-001-T01]
- [ ] Issue 2: Cosmos DB RBAC Permissions [ID=F28-11-001-T02]
- [ ] Issue 3: Mock Groups Not Working (LOCAL_DEBUG) [ID=F28-11-001-T03]

### Story: Fallback Mechanism Flow Diagram [ID=F28-11-002]

## Feature: Known Issues & Fixes [ID=F28-12]

### Story: Fix 1: JWT Group Extraction Bug (February 4, 2026) [ID=F28-12-001]

### Story: Fix 2: LOCAL_DEBUG Mock Groups (February 4, 2026) [ID=F28-12-002]

### Story: Fix 3: Lifespan Mock Groups (Already Present) [ID=F28-12-003]

## Feature: API Endpoints [ID=F28-13]

### Story: GET /getUsrGroupInfo [ID=F28-13-001]

### Story: POST /updateUsrGroupInfo [ID=F28-13-002]

## Feature: Configuration Reference [ID=F28-14]

### Story: Environment Variables [ID=F28-14-001]

### Story: Azure AD App Registration [ID=F28-14-002]

## Feature: Custom Example Prompts Per Group [ID=F28-15]

### Story: Configuration Location [ID=F28-15-001]

### Story: Schema [ID=F28-15-002]

### Story: Updating Examples [ID=F28-15-003]

## Feature: Testing & Validation [ID=F28-16]

### Story: Test Script [ID=F28-16-001]

### Story: Manual Testing Checklist [ID=F28-16-002]

- [ ] User can log in via Azure AD [ID=F28-16-002-T01]
- [ ] User sees their assigned groups in UI [ID=F28-16-002-T02]
- [ ] User can select a group from dropdown [ID=F28-16-002-T03]
- [ ] User can upload documents (Admin/Contributor only) [ID=F28-16-002-T04]
- [ ] User can search documents in their authorized index [ID=F28-16-002-T05]
- [ ] User sees custom example prompts for their group [ID=F28-16-002-T06]
- [ ] Authorization persists across browser sessions [ID=F28-16-002-T07]
- [ ] Multi-group users see correct role precedence [ID=F28-16-002-T08]

## Feature: Related Documentation [ID=F28-17]

### Story: Primary References [ID=F28-17-001]

### Story: Code References [ID=F28-17-002]

### Story: Debug Sessions [ID=F28-17-003]

## Feature: FAQ [ID=F28-18]

## Feature: Related Documentation [ID=F28-19]

### Story: Complete Documentation Suite [ID=F28-19-001]

### Story: Key Documentation Features [ID=F28-19-002]

## Feature: Change Log [ID=F28-20]
