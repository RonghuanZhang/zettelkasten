---
"type:": project-notes
"title:": 20251014134440-Hive Quality Checklist
"id:": 20251014134440
"created:": 2025-10-14T13:44:40
tags:
  - project/hive
"processed:": false
"archived:": false
---


# 1 Functional Suitability
## 1.1 Functional Completeness (None)
## 1.2 Functional Correctness
- [ ] Use-case coverage, edge cases 🔺
## 1.3 Functional Appropriateness
- [ ] Clear README doc in repository.
# 2 Interaction Capability
## 2.1 Learnability
- [ ] CLI help command. 100% commands and params description. 🔺
- [ ] Project README and guide. Ensure no outdate info. No mistaken info. 🔺 
## 2.2 User Error Protection
- [ ] Param validation: required, enums, ranges, cross-field checks. 🔺
- [ ] No internal stack traces to users; map to business error codes. 🔺 
## 2.3 Self-descriptiveness
- [ ] Unified success/warn/error templates.
## 2.4 Operability
- [ ] Config more short flags for common command. 🔺 
- [ ] CLI upgrade feature. 🔽 
# 3 Maintainability
## 3.1 Analysability
- [ ] Unify the code style.
- [ ] Trace
- [ ] Review log. 🔺 
- [ ] Request/Response/Method check. 🔺 
- [ ] Remove useless comment.
## 3.2 Modularity
- [ ] Review domain boundaries and clean TODOs. 🔺 
## 3.3 Reusability
- [ ] Detect and deduplicate code.

# 4 Security
## 4.1 Confidentiality
- [ ] Least privilege: separate tenant SA, pipeline SA, app SA with RBAC. 🔺
- [ ] NetworkPolicy enforced. 🔺 
- [ ] Container run as non-root. 🔺 
## 4.2 Accountability
- [ ] Audit log: who/when/what/result/resource ID. Non-admin cannot delete.
## 4.3 Authenticity
- [ ] Migrate from Sa-Token to Keycloak (OIDC/OAuth2). Integrate Harbor via OIDC. 🔺 
# 5 Reliability
## 5.1 Availability
- [ ] Monitor and alert rule. Config the alert channel （Slack、Email）
- [ ] Health endpoints; readiness/liveness probes.