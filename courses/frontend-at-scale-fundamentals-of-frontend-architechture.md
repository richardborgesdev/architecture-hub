# Frontend At Scale: Fundamentals of Frontend Architecture
[[course link]](https://frontendatscale.com/courses/frontend-architecture/foundations/introduction/)

## 1. Foundations
1. What's software architecture?
    1. is about the structure of a system
2. Who needs an architect? (Martin Fowler) [[article link]](https://martinfowler.com/ieeeSoftware/whoNeedsArchitect.pdf)
    1. "architecture is the things that people perceive as hard to chance"
3. Books references
    1. [Design It!](https://pragprog.com/titles/mkdsa/design-it/)
        1. "A system's software architecture is the set of significant design decisions about how the software is organized to promote desired quality attributes and other properties."
    1. [Head First Software Architecture](https://www.oreilly.com/library/view/head-first-software/9781098134341/)
        1. Four dimensions: style, characteristics, decisions, components
1. What is Frontend Architecture?
    1. style: micro-frontends | Monolithic RSC
    1. characteristics: scalability, deployability, maintanability | performance, agility, reliability
    1. decisions: share globals state with signals, compose, MFEs, client-side | Share global state with store mutate data with server actions
    1. components: Models, collections, views, templates, classes | Client components, Server components, hooks, services
1. Software design vs Architecture
    1. architecture: hard to change, strategic, higher level
    1. design: easy to change, tactical, lower level
    1. underestimating an architectural decision x overstimating a design decision
1. The Frontend Architect Role
    1. setting technical direction
    1. applying architectural thinking [[youtube link]](https://www.youtube.com/watch?v=ssAgU9eYuvQ)
        1. trade-offs
        1. business drivers to architectural requirements
        1. wide technical knowledge with some depth
    1. Glue work [[article link]](https://www.noidea.dog/glue)
        1. invisible but very important work: docs, meetings, sponsorship

## Understanding
1. Before We Start
    1. Understanding > Design > Implementation (Not the real world process)
    1. In real world the steps usually repeat multiples times (Microinteractions)
1. Project Introduction
    1. [[github link]](https://github.com/Charca/frontend-architecture-workshop/blob/main/documents/project-spec.md)
    1. System Context Diagram
1. The C4 model [[site link]](https://c4model.com/)
    1. context, containers, components, code
    1. C4 exercise 1 [[github link]](https://github.com/Charca/frontend-architecture-workshop/blob/main/exercise-solutions/container-diagram-draft.png)
1. Architectural Drivers
    1. Drivers > Requirements > Decisions
    1. Business goals, Quality Attributes, Constraints, Requirements, Team Experience + Knowledge, technology trends > Architecture
1. Architectural Requirements
    1. FullSnack Architectural Requirements Doc [[github link]](https://github.com/Charca/frontend-architecture-workshop/blob/main/documents/requirements.md)
    1. Template [[notion link]](https://charca.notion.site/Architectural-Requirements-Doc-f47fe67cd5ba408d840306e01eb38081)
    1. Solver exercise [[github link]](https://github.com/Charca/frontend-architecture-workshop/blob/main/exercise-solutions/requirements-final.md)
1. Architectural Decisions
    1. Deciding which frontend framework to use
    1. Version control
    1. Codebase organization
    1. third-party vendors
    1. Observability and monitoring
    1. Don't try to make every decision
    1. Why is more important than what
    1. ADR
        1. Template [[github link]](https://github.com/joelparkerhenderson/architecture-decision-record/blob/main/locales/en/templates/decision-record-template-by-michael-nygard/index.md)
        1. Article [[article link]](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
        1. Solved ADR exercise [[github link]](https://github.com/Charca/frontend-architecture-workshop/blob/main/documents/adr.md)

## Designing
1. Entities, Modules, and Components
    1. Entities: Model, Class / Interface / Type, Service
    1. Modules: Folder, Route / Page
    1. Componentes: View, UI Component
1. Domain Modeling [[article link]](https://frontendatscale.com/issues/25/)
    1. Method for discovering Entities and describing their Attributes and Operations
        1. Data engineering -> database schema
        1. OOP -> classes, interfaces and hierarchy
        1. Frontend -> structure align with UI
    1. Exercise [[github link]](https://github.com/Charca/frontend-architecture-workshop/blob/main/documents/domain-model.md)
        1. mermaid tool [[site link]](https://mermaid.live/)
        1. solution [[github link]](https://github.com/Charca/frontend-architecture-workshop/blob/main/exercise-solutions/domain-model-final.md)
1. Breaking Things Down
    1. Finding Modules
1. Useful diagrams
    1. Flowcharts [[site link]](https://creately.com/guides/flowchart-guide-flowchart-tutorial/)
    1. Statecharts [[site link]](https://stately.ai/docs/state-machines-and-statecharts)
    1. Class diagrams [[site link]](https://www.lucidchart.com/pages/uml-class-diagram)
    1. Sequence diagrams [[site link]](https://creately.com/guides/sequence-diagram-tutorial/)
    1. Exercise:
        1. Order Placement Sequence [[github link]](https://github.com/Charca/frontend-architecture-workshop/blob/main/documents/sequence-diagram-1.md)
        1. Solution [[github link]](https://github.com/Charca/frontend-architecture-workshop/blob/main/exercise-solutions/sequence-diagram-2.md)
1. The Design Document
    1. Stucture: overview, context, goals and non-goals, high level design, alternatives considered, detailed design, timeline, risks and open questions
    1. FullSnack high-level design doc [[github link]](https://github.com/Charca/frontend-architecture-workshop/blob/main/documents/design-doc.md)
    1. Design Docs at google [[site link]](https://www.industrialempathy.com/posts/design-docs-at-google/)
    1. Painless functional specifications [[site link]](https://www.joelonsoftware.com/2000/10/02/painless-functional-specifications-part-1-why-bother/)
    1. How to write a good software design doc [[site link]](https://www.freecodecamp.org/news/how-to-write-a-good-software-design-document-66fcf019569c/)
