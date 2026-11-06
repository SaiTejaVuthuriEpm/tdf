# Project: Generic, Annotation-Driven HTML Reporting Engine

## Role

Act as a Senior Java Automation Architect, Software Architect and Reporting Framework Developer.

Your responsibility is to design and implement a reusable, annotation-driven HTML reporting engine that can be integrated with multiple automation frameworks and projects.

The initial consumer will be a Java-based API automation framework using TestNG. However, the reporting engine must remain independent of API-specific concepts, TestNG, business logic and project-specific data structures.

The architecture must support future integration with:
- API automation.
- UI automation.
- Database testing.
- Integration testing.
- Performance or other execution-based testing frameworks.

The primary objective is to build the reporting engine once and reuse it across projects.

---

# 1. Architectural Foundation: Two-Layer Design

The implementation MUST follow a two-layer architecture.

## Layer 1: Core Reporting Engine

This is the central, reusable reporting component.

It must not depend on TestNG or any specific automation framework.

Its responsibilities include:

- Custom annotation definitions.
- Annotation discovery.
- Reflection-based field extraction.
- Test metadata collection through generic interfaces.
- Generic report data models.
- Report data snapshotting.
- Section and field organization.
- Sensitive data masking.
- HTML rendering.
- Individual execution report generation.
- Error report generation.
- Management report generation.
- Execution history persistence.
- Report configuration management.
- File and directory management.

The core reporting engine should accept generic execution and test result data through well-defined interfaces or data models.

It must not contain:
- TestNG-specific listeners.
- TestNG-specific result handling.
- API request or response assumptions.
- GCP-specific processing.
- Metric-specific processing.
- Project-specific business logic.

## Layer 2: Integration Adapters

Integration adapters connect individual automation frameworks to the core reporting engine.

The first adapter will be a TestNG integration adapter.

Its responsibilities include:

- Listening to TestNG lifecycle events.
- Identifying test methods and test classes.
- Discovering custom test annotations.
- Capturing test start, completion, success, failure and skip events.
- Submitting annotated POJOs to the core reporting engine.
- Associating test results with the correct execution.
- Triggering report finalization at the appropriate lifecycle stage.

Future adapters should be possible without changing the core reporting engine.

Examples:
- TestNG adapter.
- JUnit adapter.
- Custom API automation adapter.
- Other framework-specific adapters.

### Architectural principle

The dependency direction must always be:

Automation Framework → Integration Adapter → Core Reporting Engine

The core engine must never depend on the integration adapter.

Use appropriate interfaces and dependency inversion to maintain this separation.

---

# 2. First Step: Inspect the Existing Project

Before implementing anything:

1. Inspect the existing project structure.
2. Identify Java version and build tool.
3. Identify existing TestNG configuration and listeners.
4. Identify test execution lifecycle.
5. Identify existing reporting implementations.
6. Identify POJOs and object models.
7. Identify execution orchestration and run ID generation.
8. Identify existing logging, JSON and file utilities.
9. Identify existing dependencies that can be reused.
10. Identify any parallel execution configuration.

Do not assume existing class names, method signatures or implementation details.

Prepare a concise architectural assessment and implementation plan before modifying code.

The implementation must fit the existing project instead of introducing an unrelated framework.

---

# 3. Required Folder Structure

Use the following structure:

```text
Automation Framework/
└── RuntimeHistory/
    ├── run-history.json
    │
    ├── RUN-20261106-001/
    │   ├── ManagementReport.html
    │   │
    │   └── Run-20261106_123456/
    │       ├── index.html
    │       └── errors/
    │           ├── error_<uuid1>.html
    │           └── error_<uuid2>.html
    │
    ├── RUN-20261106-002/
    │   ├── ManagementReport.html
    │   │
    │   └── Run-20261107_123456/
    │       ├── index.html
    │       └── errors/
    │           ├── error_<uuid1>.html
    │           └── error_<uuid2>.html
    │
    └── ...
```

### Folder responsibilities

**RuntimeHistory**
- Root reporting directory.
- Contains central execution history JSON.
- Contains individual execution cycle folders.

**run-history.json**
- Maintains lightweight execution metadata.
- Must preserve historical entries.
- Must not contain complete HTML reports, payloads or stack traces.

**RUN-<runId>**
- Represents an execution cycle.
- Contains its consolidated `ManagementReport.html`.
- Contains one or more individual execution directories where applicable.

**Run-<timestamp>**
- Represents an individual test execution.
- Contains one `index.html` covering all test cases in that execution.
- Contains its own `errors/` directory.

**errors/**
- Contains separate detailed error HTML pages.
- All error pages must belong to their respective individual execution.
- Never mix error reports from different executions.

Keep `runId`, `executionId` and individual test case identity separate.

Do not confuse the central `run-history.json` file with individual run folders.

---

# 4. Core Reporting Engine: Package Design

Create a clean package structure aligned with the existing project conventions.

Suggested logical organization:

```text
reporting/
│
├── annotations/
│   ├── ReportField.java
│   ├── TestDescription.java
│   └── supporting annotations, if required
│
├── core/
│   ├── ReportingEngine.java
│   ├── ReportCollector.java
│   ├── ReportDataProcessor.java
│   └── ReportFinalizer.java
│
├── discovery/
│   ├── AnnotationDiscoveryService.java
│   ├── FieldResolver.java
│   └── ReportObjectExtractor.java
│
├── model/
│   ├── ExecutionReport.java
│   ├── TestCaseReport.java
│   ├── ReportSection.java
│   ├── ReportFieldData.java
│   ├── ErrorReport.java
│   └── ExecutionHistoryEntry.java
│
├── generator/
│   ├── HtmlReportGenerator.java
│   ├── ErrorReportGenerator.java
│   └── ManagementReportGenerator.java
│
├── history/
│   ├── RunHistoryRepository.java
│   └── RunHistoryManager.java
│
├── configuration/
│   ├── ReportConfiguration.java
│   └── ReportConfigurationLoader.java
│
├── filesystem/
│   ├── ReportDirectoryManager.java
│   └── ReportPathResolver.java
│
├── integration/
│   └── testng/
│       ├── TestNGReportingListener.java
│       ├── TestNGExecutionAdapter.java
│       └── TestNGMetadataResolver.java
│
└── resources/
    ├── templates/
    │   ├── execution-report template
    │   ├── error-report template
    │   └── management-report template
    │
    ├── css/
    └── js/
```

This is a suggested structure, not a requirement to create unnecessary classes. Adapt it to the existing codebase while preserving the separation of responsibilities.

---

# 5. Custom Annotation Design

## 5.1 `@ReportField`

Create a generic field-level annotation.

Example:

```java
public class CustomerApiReportData {

    @ReportField(
        label = "API Name",
        section = "API Information",
        order = 1
    )
    private String apiName;

    @ReportField(
        label = "Endpoint",
        section = "Request",
        order = 2
    )
    private String endpoint;

    @ReportField(
        label = "HTTP Method",
        section = "Request",
        order = 3
    )
    private String method;

    private String internalReference;
}
```

Only explicitly annotated fields must be displayed.

`internalReference` must not appear.

Suggested annotation attributes:

- `label`
- `section`
- `order`
- `description`
- `display`
- `sensitive`
- `format`

Requirements:

- Runtime retention.
- Field target.
- Support private fields.
- Support inherited annotated fields.
- Support primitive types, strings, enums, dates, arrays, collections and maps.
- Support nested POJOs safely.
- Prevent circular references and infinite recursion.
- Support field ordering.
- Group fields by section.
- Handle null values using configurable behavior.
- Never expose unannotated fields through automatic serialization or `toString()`.

The annotation must remain generic. Do not add API-specific attributes.

## 5.2 `@TestDescription`

Create a method-level annotation.

Example:

```java
@Test
@TestDescription(
    value = "Verify successful customer creation using a valid request"
)
public void createCustomerTest() {
    // Test implementation
}
```

Requirements:

- Runtime retention.
- Method target.
- Readable test description.
- Graceful handling when missing.
- Compatible with the TestNG integration adapter.
- No dependence on test method naming conventions.

## 5.3 Extensibility

Design the annotation mechanism to allow future annotation types without rewriting the HTML generator.

Keep the first implementation lean.

Additional test metadata such as category, severity, tags and owner can be considered where useful, but should not be mandatory.

---

# 6. Annotation Discovery and Field Extraction

Implement a reusable discovery and extraction mechanism in the core engine.

Responsibilities:

1. Discover test method metadata through the adapter-supplied metadata.
2. Extract `@TestDescription`.
3. Accept one or multiple POJOs associated with a test.
4. Discover all `@ReportField` annotations.
5. Extract only annotated fields.
6. Group fields by section.
7. Apply annotation ordering.
8. Convert complex values to readable representations.
9. Mask sensitive values.
10. Handle nulls, empty objects and reflection errors.
11. Support nested POJOs and collections safely.

Important requirements:

- No TestNG dependencies inside the discovery service.
- No API-specific assumptions.
- Support multiple report data objects per test.
- Preserve object context when multiple POJOs use the same section name.
- Capture immutable or safely detached snapshots so that later mutations do not alter already captured report data.
- Support parallel execution.
- Avoid unsafe reflection and uncontrolled object serialization.

Use a clearly defined generic reporting API to allow consuming projects to submit data.

---

# 7. Generic Reporting API

Provide a clean public API for consuming projects.

The API should conceptually support:

- Starting an execution.
- Starting a test case.
- Attaching one or multiple POJOs.
- Capturing test descriptions.
- Recording test success, failure or skip.
- Associating exceptions with test cases.
- Finalizing an execution.
- Generating reports.

The exact interface and method names should be determined after inspecting the existing codebase.

A consuming project should not need to know how HTML is generated or how report files are stored.

Example of intended usage:

```java
@Test
@TestDescription("Verify successful customer creation")
public void createCustomerTest() {

    CustomerApiReportData requestData = new CustomerApiReportData();

    // Populate request information.

    reportingEngine.attach(requestData);

    // Execute API request.

    CustomerResponseReportData responseData =
        new CustomerResponseReportData();

    // Populate response and comparison information.

    reportingEngine.attach(responseData);
}
```

This is illustrative only. Implement a real, thread-safe integration API and provide actual usage examples matching the completed implementation.

Consider whether a test-scoped reporting context or execution context is appropriate to prevent data from one test leaking into another.

---

# 8. Generic Report Data Models

Separate data collection from HTML rendering.

Suggested models:

### ExecutionReport
- runId
- executionId
- executionName
- startTime
- endTime
- duration
- overallStatus
- totalTests
- passedTests
- failedTests
- skippedTests
- testCases

### TestCaseReport
- testName
- testDescription
- testClass
- category/tags (optional)
- status
- startTime
- endTime
- duration
- reportSections
- errorReferences

### ReportSection
- sectionName
- objectName/context
- fields

### ReportFieldData
- label
- value
- order
- displayFormat
- sensitive
- description

### ErrorReport
- errorId
- testName
- testDescription
- failureType
- errorMessage
- stackTrace
- timestamp
- associated report sections

These models are conceptual. Use suitable types, immutability and serialization conventions.

The core data model must not assume API-specific fields.

---

# 9. Individual Execution Report: `index.html`

Generate one HTML page per individual execution.

All test cases from that execution must be present in this page.

## Execution summary

Display:

- Execution ID.
- Execution start and end time.
- Total duration.
- Overall status.
- Total test cases.
- Passed count.
- Failed count.
- Skipped count.
- Pass percentage.

## Test case listing

Provide a searchable and filterable table with:

- Test Name.
- Test Description.
- Status.
- Duration.
- Start Time.
- Details / Actions.

## Expandable test case details

Each test case must have an expandable/collapsible detail view.

Display:

- Test metadata.
- Test description.
- Discovered annotated sections.
- Annotated fields.
- Failure details and links, when applicable.

Do not hardcode sections such as Request, Response, Validation or API Information.

These should appear only when the consumer supplies annotated data.

## User interface

- Modern, professional and responsive layout.
- Search and status filters.
- Expand/collapse details.
- Clear pass/fail/skip indicators.
- Readable JSON formatting where appropriate.
- Collapsible large payloads.
- HTML escaping and safe content rendering.
- No external internet dependency for core report functionality.
- Local or self-contained CSS/JavaScript assets.
- Good usability with hundreds of test cases.

---

# 10. Error Report: `errors/error_<uuid>.html`

Generate a separate HTML error page for every failed test or independently reportable assertion failure that can reliably be identified.

Location:

`RuntimeHistory/RUN-<runId>/Run-<timestamp>/errors/error_<uuid>.html`

Display:

- Error ID.
- Test name.
- Test description.
- Failure timestamp.
- Failure type.
- Error message.
- Full stack trace.
- Associated annotated report sections.
- Relevant request/response information if supplied.
- Expected versus actual information if supplied.
- Link back to the individual `index.html`.

Requirements:

- UUID-based filenames.
- Errors must remain within the corresponding execution directory.
- The test case in `index.html` must link to its error report.
- Handle large stack traces and payloads.
- Do not duplicate unnecessary data.
- Error report generation failures must not replace or hide the original test failure.
- Preserve the original TestNG result and exception.

---

# 11. Management Report: `ManagementReport.html`

The management report is a consolidated execution-history dashboard.

It must help managers, architects and engineers understand execution trends without opening individual reports.

Location:

`RuntimeHistory/RUN-<runId>/ManagementReport.html`

Use `run-history.json` as the central history source.

## Dashboard summary

Display:

- Total execution cycles.
- Total individual executions.
- Latest execution status.
- Latest execution timestamp.
- Total tests across available executions.
- Overall pass/fail trends.

## Execution history table

Show latest executions first.

Suggested columns:

- Run ID.
- Execution ID.
- Execution Date.
- Start Time.
- Duration.
- Total Tests.
- Passed.
- Failed.
- Skipped.
- Pass Percentage.
- Status.
- Report Link.

## Trend visualizations

Provide:

- Pass percentage trend.
- Passed versus failed test count trend.
- Execution duration trend.
- Execution status history.

Do not display misleading charts when insufficient historical data exists.

## Navigation

- Every execution entry must link to its corresponding individual `index.html`.
- Management reports must remain accessible from their respective run cycle folders.
- Historical execution links must continue to work.

## Historical data handling

- Never overwrite previous execution records.
- A new execution must add a new history entry.
- Preserve historical information when generating a new management report.
- Handle missing report folders gracefully.
- Handle history records whose report files are no longer present.
- Reconcile history metadata with the actual report directory structure when necessary.

The management report is a history and trend view, not merely a summary of the latest execution.

---

# 12. Configuration Support

Support both annotations and a configuration file.

Annotations control:
- Which fields are reportable.
- Field labels.
- Sections.
- Field ordering.
- Field-level display and masking behavior.

Configuration controls presentation and general report behavior.

Suggested properties:

- Project name.
- Report title.
- Project description.
- Logo or logo path.
- Theme.
- Primary and secondary colors.
- Timestamp format.
- Null-value display policy.
- Default sensitive-value masking policy.
- Maximum payload display length.
- JSON formatting behavior.
- Chart enablement.
- Stack trace expansion behavior.
- History retention policy.
- Default field ordering behavior.

Use YAML, JSON or properties based on existing project conventions.

Provide sensible defaults.

Important: Configuration must not expose fields that have not been explicitly annotated.

The design must allow a project to override presentation settings without modifying core reporting code.

---

# 13. TestNG Integration Adapter

Implement TestNG-specific functionality only in the integration layer.

Responsibilities:

- Identify execution lifecycle.
- Identify individual test lifecycle.
- Discover method-level `@TestDescription`.
- Capture test class and method metadata.
- Capture success, failure and skipped outcomes.
- Capture execution timings.
- Associate exceptions with the correct test.
- Associate attached POJOs with the correct test.
- Finalize execution reports.
- Trigger management report generation.
- Persist execution history.

Consider the appropriate TestNG interfaces, listeners and lifecycle hooks based on the actual project.

Ensure:

- One `index.html` per individual execution.
- All test cases are collected into the correct report.
- Errors are written to the correct execution's `errors/` directory.
- Parallel test execution is supported.
- Execution contexts are isolated.
- Report generation does not alter test results.
- Interrupted executions can be represented appropriately.

The TestNG adapter must delegate reporting work to the core engine.

Do not put HTML generation or business-specific logic in the TestNG listener.

---

# 14. History Persistence

Implement a robust repository for `run-history.json`.

Requirements:

- Read existing history.
- Create a new history record.
- Update in-progress execution metadata.
- Persist final execution counts and timings.
- Preserve previous history.
- Handle missing or empty files.
- Handle malformed JSON safely.
- Never silently discard existing history.
- Use safe or atomic file writes.
- Handle concurrent updates.
- Store relative report paths wherever practical.
- Keep the JSON lightweight.

Do not store complete request/response payloads or stack traces in this file.

---

# 15. API Automation Consumer Example

Demonstrate how the generic reporting engine will be used by the initial API automation project.

Example:

```java
public class CustomerApiReportData {

    @ReportField(
        label = "API Name",
        section = "API Information",
        order = 1
    )
    private String apiName;

    @ReportField(
        label = "Endpoint",
        section = "Request",
        order = 2
    )
    private String endpoint;

    @ReportField(
        label = "HTTP Method",
        section = "Request",
        order = 3
    )
    private String method;

    @ReportField(
        label = "Request Body",
        section = "Request",
        order = 4
    )
    private Object requestBody;

    @ReportField(
        label = "Status Code",
        section = "Response",
        order = 1
    )
    private int statusCode;

    @ReportField(
        label = "Response Body",
        section = "Response",
        order = 2
    )
    private Object responseBody;

    @ReportField(
        label = "Comparison Result",
        section = "Validation",
        order = 1
    )
    private String comparisonResult;
}
```

Provide a complete working TestNG example demonstrating:

1. Test description.
2. Request data creation.
3. Request data attachment.
4. API invocation.
5. Response data creation.
6. Response data attachment.
7. Comparison/validation data attachment.
8. Successful execution reporting.
9. Failed execution reporting.
10. Error page generation.

The reporting engine must not contain these API fields as predefined properties.

---

# 16. Quality, Reliability and Security

Follow:

- SOLID principles.
- Separation of concerns.
- Dependency inversion.
- Thread safety.
- Low coupling.
- High cohesion.
- Maintainable naming conventions.
- Clear logging.
- Robust exception handling.
- Safe filesystem operations.
- Immutable snapshots where appropriate.

Security requirements:

- Escape HTML content.
- Prevent script injection through test data.
- Mask sensitive annotated fields.
- Avoid exposing unannotated fields.
- Avoid logging sensitive payloads unnecessarily.
- Avoid uncontrolled recursive serialization.
- Treat generated HTML as potentially sensitive test output.

Keep HTML generation independent of TestNG.

The core engine should be usable by another framework through a generic reporting interface.

---

# 17. Validation and Automated Tests

Create unit and integration tests covering at least:

1. Annotated field discovery.
2. Exclusion of unannotated fields.
3. Private field extraction.
4. Inherited annotated fields.
5. Custom labels.
6. Section grouping.
7. Field ordering.
8. Test description discovery.
9. Multiple POJOs per test.
10. Null and empty values.
11. Sensitive field masking.
12. Nested objects and collections.
13. Circular-reference protection.
14. Successful test reporting.
15. Failed test reporting.
16. Skipped test reporting.
17. Error report generation.
18. Correct error report links.
19. Execution isolation.
20. Multiple historical executions.
21. History JSON preservation.
22. Management report generation.
23. Parallel execution.
24. Interrupted execution handling.
25. HTML escaping and injection protection.

---

# 18. Deliverables

Provide a complete implementation with:

1. Core reporting engine.
2. Integration adapter.
3. Custom annotations.
4. Generic report models.
5. Annotation discovery utilities.
6. Generic reporting API.
7. TestNG lifecycle integration.
8. Individual HTML generator.
9. Error HTML generator.
10. Management HTML generator.
11. History repository.
12. Configuration loader and sample configuration.
13. HTML/CSS/JavaScript templates or assets.
14. Unit and integration tests.
15. API automation usage example.
16. Updated dependency configuration, if necessary.
17. Exact package structure and file placement.
18. Execution instructions.
19. Sample `run-history.json`.
20. Sample generated folder structure.
21. Complete change summary.

---

# 19. Implementation Workflow and Constraints

Follow this workflow:

### Phase 1: Project inspection
Inspect the existing codebase and report:
- Existing architecture.
- Relevant existing classes.
- Current lifecycle.
- Reusable components.
- Identified gaps.

### Phase 2: Architecture and implementation plan
Provide:
- Proposed two-layer architecture.
- Class responsibilities.
- Integration points.
- Files to add.
- Files to modify.
- Dependency changes, if any.
- Potential compatibility risks.

Wait for approval before making modifications.

### Phase 3: Complete implementation
Once approved:
- Implement the complete core engine.
- Implement the TestNG adapter.
- Integrate with the existing execution flow.
- Add configuration.
- Add report templates.
- Add tests.
- Preserve existing functionality.

### Phase 4: Validation
Run:
- Compilation.
- Unit tests.
- Integration tests.
- A sample successful execution.
- A sample failed execution.
- Multiple execution cycles.
- Parallel execution validation, where supported.

### Phase 5: Final handover
Provide:
- Complete list of added and modified files.
- Exact file paths.
- Architectural explanation.
- Configuration instructions.
- Annotation usage examples.
- Execution instructions.
- Sample output structure.
- Validation results.
- Known limitations, if any.

## Final Non-Negotiable Requirements

1. Core Reporting Engine and Integration Adapter must remain separate.
2. The core engine must not depend on TestNG.
3. The core engine must not contain API-specific or business-specific fields.
4. Only annotated POJO fields may be displayed.
5. Test methods must support `@TestDescription`.
6. One individual execution must have one `index.html`.
7. Each individual execution must have its own `errors/` folder.
8. Management reports must show historical execution trends.
9. Historical records must never be unintentionally overwritten.
10. Configuration and annotations must work together.
11. Parallel executions must not mix data.
12. Existing framework behavior must be preserved.
13. The solution must be reusable in other projects without modifying the core engine.
14. Deliver a complete, coherent implementation rather than scattered code snippets.

**Success criterion:** A new automation project should be able to adopt this reporting engine by adding the core reporting dependency, implementing or using the appropriate integration adapter, annotating its test methods and POJO fields, and configuring its presentation—without changing the reporting engine itself.