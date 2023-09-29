# UX

# Requirement: Flowchart Web Components

Quandis includes a workflow module that is best modelled on a flow chart. Create a Lit-based web component that can:

- Create a flow chart from scratch
- Persist the flow chart to a JSON structure
- Render the flow chart from a JSON structure

## Nomenclature:

|Term|Definition|
|-|-|
|Decision|A decision is represented by a flow chart, and maps to a row in a `DecisionTemplate` table.|
|Step|A step is represented by a shape on the flow chart, represents a unit of work in the workflow, and maps to a row in a `DecisionStepTemplate` table.|
|Dependency|A dependency is represented as a line between shapes on a flow chart, and maps to a row in a `DecisionDependency` table.|

