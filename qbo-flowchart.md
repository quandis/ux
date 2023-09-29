# Flowchart Web Component

Quandis includes a workflow module that is best modelled on a flow chart. Create a Lit-based web component named `qbo-flowchart` that can:

- Create a flow chart from scratch
- Persist the flow chart to a JSON structure
- Render the flow chart from a JSON structure

The web component must run in Chrome, Edge, Firefox and Safari, and any dependencies must be available as Node packages.

## Functionality

The web componet must support the following concepts:

- Add any one of a variety of shapes (see nomenclature below for types of shapes)
- Remove a shape
- Add a dependency (line) between shapes
- Remove a dependency
- Raise a custom event when a shape is clicked on
- Raise a custom event when a line is clicked on
- Display text within a shape (all text will be part of the serialized flow chart)
  - use a slot (if practical) to control the content of the text of the shape
  - e.g. we may wish to show the name of the step, and how long it takes

## Nomenclature:

|Term|Definition|
|-|-|
|`Decision`|A decision is represented by a flow chart, and maps to a row in a `DecisionTemplate` table.|
|`Step`|A step is represented by a shape on the flow chart, represents a unit of work in the workflow, and maps to a row in a `DecisionStepTemplate` table.|
|`Dependency`|A dependency is represented as a line between shapes on a flow chart, and maps to a row in a `DecisionDependency` table.|
|`Document`|A step that creates a document (mail merge).|
|`Task`|A step that creates a task for a user to complete.|
|`Message`|A step that creates a message (email, SMS text, or 'yellow sticky note').|
|`Score`|A step that creates a score (think Excel snapshot with formulas applied).|
|`Ledger`|A step that creates a ledger (invoice, bill, or other collection of accounting-like line items).|
|`Process`|A step that creates a process (think manilla folder inside an account folder in a filing cabinet).|
|`If/Then`|A step that evaluates data, outputting true or false.|
|`Polling`|A step that evaluates data on a recurring bases, until the data condition is met.|
|`Advanced Step`|A step that invoke any API endpoint configured in the system.|

## Shapes

The flowchart should support a different series of shapes (defined above). At the moment, our UI includes the following icons associated with each step. 

![image](https://github.com/quandis/ux/assets/1129020/4e6b1f4c-5590-4842-8a35-cda7d496c4f4)

The rendering of each shape should be easy to modify over time (perhaps using named slots).
Existing QBO sites use Bootstrap for our css framework. 
- If adopting Bootstrap is useful, please feel free to do so.
- If custom css classes are being created, they should match Bootstrap classes as closely as possible

## Edge case considerations

- Workflows average about 10 steps (shapes), but some may comprise 100 steps or more.
  - A zooming feature would be nice, but is not required
- Dependencies: `and` vs `or`
  - `or` dependencies are represented by the same `DecisionDependency.Group`: A or B or C is represented by `A.Group = B.Group = C.Group`
  - `and` dependencies are represented by different `DecisionDependency.Group`: A and B is represented by `A.Group != B.Group`
  - we suggest that `or` dependencies map into the same point on a dependent shape, and `and` dependencies map into different points on a dependent shape. However, if it's easier to create `or` and `and` shapes, that is acceptable.
- We suggest automatically including `Start` and `End` shapes, even though there is no `DecisionStepTemplate` representing these concepts.
- The flow chart must be able to render a workflow that was created or modified without the flow chart.
  - If a user adds a `DecisionStepTemplate` outside of the flowchart, QBO will inject this new step into the serialized flow chart the next time it renders. 
  - This implies there may be no "coordinate positioning" associated with a step when the flow chart renders
  - The rendering of steps without "coordinate positioning" need not be pretty, but it must to break other functionality (e.g complete cover existing steps such that they cannot be clicked on)

## Serilization

You may design whatever serialization format is needed, provided it can be precisely translated to our `DecisionStepTemplate` and `DecisionDependency` data structures. Quandis will assume responsibility for persisting the serialize flow chart, and modifying the serialized flow chart when steps are added outside the flowchart web component. Assume that the flowchart web component can make an API call to retrieve a serialized instance. 

## Testing

The web component must be testable with [web-test-runner](https://modern-web.dev/docs/test-runner/overview/)
