
---

Enterprise Engineering Intelligence Kit (EEIK)

Vision 1.0 — Enterprise Project Bootstrap Platform

---

Executive Summary

EEIK is not an agent repository.

EEIK is not a prompt repository.

EEIK is not a collection of Claude configuration files.

EEIK is an Enterprise Project Bootstrap Platform that assembles the right engineering intelligence for a specific project.

The goal is to create:

- Faster project startup
- Consistent engineering standards
- Reduced architectural drift
- Reduced governance overhead
- Reusable enterprise intelligence
- Dynamic agent generation
- Project-specific AI operating systems

Instead of maintaining hundreds of static agents, EEIK maintains reusable capability packs and a bootstrap engine that composes a project-specific workspace.

---

Core Philosophy

Traditional Approach

Repository
 ├─ 200 Agents
 ├─ 100 Prompts
 ├─ 50 Commands
 └─ 50 Hooks

Problems:

- High maintenance
- Prompt drift
- Duplicate responsibilities
- Poor discoverability
- Difficult governance

EEIK Approach

Repository
 ├─ Bootstrap Engine
 ├─ Capability Packs
 ├─ Standards Library
 ├─ Templates
 ├─ Knowledge Assets
 └─ Dynamic Agent Factory

Result:

New Project
      ↓
Bootstrap Interview
      ↓
Project Manifest
      ↓
Scaffolded Workspace
      ↓
Relevant Agents Only

---

Vision

Every new project starts with a guided discovery process.

EEIK asks:

- What are you building?
- What technologies are involved?
- What architecture style is required?
- What governance is required?
- What AI capabilities are needed?
- Is modernization involved?

Based on the answers, EEIK generates:

- Repository structure
- Claude configuration
- GitHub configuration
- Standards
- Commands
- Workflows
- Agents
- Documentation
- Governance artifacts

No unnecessary assets are created.

---

EEIK Components

1. Bootstrap Engine

Purpose:

Generate a project-specific engineering workspace.

Input:

Project questionnaire.

Output:

Project manifest.

Responsibilities:

- Requirement discovery
- Capability selection
- Agent selection
- Standards selection
- Workflow selection
- Repository generation

---

2. Capability Packs

Purpose:

Reusable building blocks.

Examples:

architecture
java
python
aws
frontend
insurance
modernization
governance
operations
ai-engineering

Each pack contains:

agents
commands
prompts
standards
templates
knowledge
examples

---

3. Dynamic Agent Factory

Purpose:

Generate new agents when required.

Instead of:

200 predefined agents

Use:

Agent Generation

Pattern:

Need identified
      ↓
Agent blueprint created
      ↓
Prompt generated
      ↓
Capabilities assigned
      ↓
Agent registered

Example:

Project requires:

Claims Settlement Optimization

No agent exists.

Agent Factory generates:

claims-settlement-specialist

automatically.

---

4. Knowledge System

Stores:

Business Rules
Events
APIs
ADR
RFC
Incidents
Lessons Learned

Purpose:

Organizational intelligence accumulation.

---

5. Governance System

Provides:

Architecture Reviews
Security Reviews
AI Reviews
Compliance Reviews
Production Readiness Reviews

Automatically enabled based on project type.

---

Repository Structure

EEIK/
│
├── bootstrap/
│
├── capability-packs/
│
├── domain-packs/
│
├── modernization-packs/
│
├── templates/
│
├── standards/
│
├── generators/
│
├── docs/
│
├── .github/
│
└── .claude/

---

Bootstrap Workflow

Step 1

Run:

/bootstrap

---

Step 2

Interview

Questions include:

Project Type

Greenfield
Modernization
MVP
PoC
Enterprise Platform
Agent Platform

Domain

Insurance
Banking
Healthcare
Retail
Generic

Backend

Java 21
Java 25
Python
Node
Mixed

Frontend

React
Angular
None

Cloud

AWS
Azure
GCP
Hybrid

Architecture

Monolith
Modular Monolith
Microservices
Serverless
Event Driven
Agentic

AI Usage

No AI
RAG
Single Agent
Multi-Agent
Enterprise Agent Platform

Modernization

No
IBM i
COBOL
Mainframe
Mixed

---

Step 3

Generate Manifest

Example:

project:
  name: claims-modernization

domain:
  insurance

backend:
  java21

frontend:
  react

cloud:
  aws

architecture:
  microservices

ai:
  langgraph

modernization:
  ibmi

---

Step 4

Capability Selection

Automatically loads:

insurance-pack
java-pack
aws-pack
react-pack
modernization-pack
ai-engineering-pack

---

Step 5

Workspace Generation

Creates:

.github
.claude
docs
standards
templates

---

Skills

Skills are higher-level workflows.

---

Project Bootstrap Skill

Purpose:

Create project workspace.

---

Architecture Discovery Skill

Purpose:

Generate architecture baseline.

---

Modernization Discovery Skill

Purpose:

Analyze legacy landscape.

---

AI Solution Discovery Skill

Purpose:

Generate agentic architecture.

---

Governance Setup Skill

Purpose:

Generate governance artifacts.

---

Delivery Setup Skill

Purpose:

Generate backlog structure.

---

Repository Scaffolding Skill

Purpose:

Create project repository.

---

Agent Factory Skill

Purpose:

Generate missing agents.

Inputs:

Need
Responsibilities
Inputs
Outputs
Constraints

Output:

Agent Specification
Prompt
Memory Strategy
Command Set

---

Capability Recommendation Skill

Purpose:

Recommend packs based on project context.

---

Knowledge Seeder Skill

Purpose:

Generate initial knowledge assets.

---

Core Agents

The platform starts with a small set.

---

Bootstrap Architect

Responsible for:

- Project discovery
- Capability selection
- Workspace generation

---

Capability Selector

Responsible for:

- Pack selection
- Dependency resolution

---

Agent Factory

Responsible for:

- Agent generation
- Agent evolution

---

Knowledge Architect

Responsible for:

- Knowledge organization
- Knowledge seeding

---

Governance Architect

Responsible for:

- Governance setup
- Review configuration

---

Solution Architect

Responsible for:

- Architecture generation

---

Delivery Architect

Responsible for:

- Delivery setup

---

Modernization Architect

Responsible for:

- Legacy modernization planning

---

AI Architect

Responsible for:

- Agentic architecture generation

---

Adoption Strategy

Phase 1

Build EEIK Core

Deliverables:

Bootstrap Skill
Manifest Model
Capability Packs

---

Phase 2

Build Core Packs

Architecture
Java
AWS
AI Engineering
Governance

---

Phase 3

Build Domain Packs

Insurance
Banking
Healthcare

---

Phase 4

Build Dynamic Agent Factory

Capabilities:

Generate Agents
Generate Commands
Generate Prompts
Generate Reviews

---

Phase 5

Build Repository Generator

Output:

Complete Project Starter Repository

---

Success Criteria

A new project should require:

/bootstrap

and approximately:

10-15 minutes

to generate:

- Architecture baseline
- Repository structure
- Claude workspace
- GitHub workflows
- Standards
- Initial agents
- Governance setup

without manual engineering setup.

---

End State

EEIK becomes:

Enterprise Engineering Operating System

rather than:

Agent Collection

The repository evolves from storing agents to generating engineering intelligence on demand.
