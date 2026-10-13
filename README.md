# Data Platform for Secondary Use of Clinical Studies Data

## Overview

The Data Platform for Secondary Use of Clinical Studies Data will make
Orion Pharma's clinical trial datasets available to R&D scientists for
internal research beyond their current use in GxP analyses and
regulatory submissions.

The initial platform will focus on SDTM and ADaM datasets from ongoing
and concluded clinical trials. It will provide a common, discoverable
data foundation that R&D scientists can access from the existing R&D
Databricks Workspace (RDDW).

The platform is intended to support three capabilities over time:

-   **Discover data:** Browse and search available clinical datasets by
    study, snapshot, and SDTM/ADaM domain.
-   **Analyze data:** Access datasets from Databricks notebooks for
    visualization and computational analysis.
-   **Use ready-made experiences:** Consume dashboards and applications
    built for agreed research use cases. This capability is planned for
    a later release.

The first release will prioritize reliable ingestion, straightforward
data organization, documentation, and access from RDDW. Cross-study data
models and purpose-built visualizations will be considered in subsequent
releases.

## Data Types

### Initial scope

The first release will store only:

-   **SDTM (Study Data Tabulation Model)** datasets.
-   **ADaM (Analysis Data Model)** datasets.

The datasets may originate from ongoing or concluded clinical trials.
They will be ingested as periodic snapshots, reflecting the data
available at the time of each quarterly ingestion.

The initial scope is intentionally narrow. Additional data types or
related assets may be considered later if a clear business benefit is
identified.

### Data classification and access assumptions

The current project hypothesis is that Orion Pharma's Legal team will
clear internal secondary use of the relevant clinical datasets across
Orion Pharma R&D functions. The intended experience is broad,
self-service access for onboarded RDDW users, without a separate request
for each dataset.

Source datasets are received in pseudonymized form. The initial proposal
is to retain that form without requiring additional full anonymization
as a default prerequisite.

**This access model is subject to formal confirmation by Legal and,
where applicable, Data Protection and other governance stakeholders.**
Pseudonymized data may remain personal data under applicable
data-protection law. Access, permitted purposes, and safeguards must
therefore follow the approved legal and governance conditions.

## Architecture

### High-level architecture

The platform separates clinical-data ingestion and organization from the
environment where R&D scientists perform their analyses.

1.  **Source systems:** Clinical datasets are transferred from either:
    -   `biowork`, an internal network share.
    -   Domino Clinical, a SaaS data platform provided by Domino Data
        Lab.
2.  **Clinical Data Platform workspace(s):** Azure Databricks
    workspace(s) are responsible for ingestion, basic checks, and
    organizing the datasets.
3.  **Cloud storage and Unity Catalog:** The ingested datasets are
    stored in cloud storage and exposed through a Unity Catalog
    catalogue.
4.  **R&D Databricks Workspace (RDDW):** The clinical catalogue is
    shared with RDDW. Scientists browse and query the data using
    Databricks, including notebooks.

``` text
Biowork ───────────────┐
                       ├──> Quarterly manual transfer
Domino Clinical ───────┘              |
                                      v
                         Clinical Data Platform
                         - Ingestion
                         - Basic ingestion checks
                         - Dataset organization
                                      |
                                      v
                         Cloud storage + Unity Catalog
                         - SDTM and ADaM datasets
                         - Study and snapshot metadata
                                      |
                              Shared catalogue
                                      |
                                      v
                         R&D Databricks Workspace
                         - Browse and query datasets
                         - Notebooks for analysis
```

### Ingestion approach

For the first release, ingestion will run quarterly. Available clinical
datasets will be transferred manually from the source system to the
platform's cloud storage.

Automation may be introduced after the first release. The initial design
should nevertheless record enough information to make each ingestion
traceable, including the source, study, snapshot or ingestion date, and
dataset identity.

### Initial storage model: Bronze layer

Release 1 will contain a **Bronze layer only**. Its purpose is to store
the ingested SDTM and ADaM datasets in a coherent, discoverable
structure while avoiding premature transformations or cross-study
harmonization.

The intended business hierarchy is:

1.  Studied compound (for example, `ODM-208`).
2.  Clinical study (for example, `CYPYDES Part1 - 3124001`).
3.  Time snapshot.
4.  Individual SDTM or ADaM dataset, represented as a table.

Unity Catalog provides a catalog--schema--table hierarchy. The exact
mapping of the business hierarchy to catalogs, schemas, tables, and
snapshot identifiers remains to be designed. The implementation should
make the structure understandable and maintainable without creating an
unnecessarily complex catalog hierarchy.

At minimum, metadata should make it possible to identify the compound,
study, snapshot, dataset name, data type (SDTM or ADaM), source, and
ingestion date. The metadata representation can be finalized during
detailed design.

### Basic ingestion checks and documentation

Release 1 will include lightweight checks to detect obvious ingestion
problems without attempting full SDTM/ADaM conformance validation or
broad data transformation.

Initial checks should be agreed during implementation and may include:

-   Confirming expected files and datasets are present.
-   Confirming files are readable and in an expected format.
-   Checking that datasets can be loaded and that their basic structure
    is available.
-   Detecting obvious duplicate dataset deliveries or snapshot
    conflicts.
-   Recording basic outcomes, such as dataset counts and ingestion
    status.

The checks and their results should be documented. Documentation should
also explain the catalogue structure, how snapshots are represented, how
users browse and query the datasets, and whom to contact for platform
support.

### Future architecture

From Release 2 onwards, the team will assess how to organize and relate
data across studies. This may include common domains, endpoints,
entities, and ways to group studies for research. The design should be
driven by concrete use cases rather than assuming a cross-study model
before its requirements are understood.

Dashboards, visualizations, and dedicated applications for agreed use
cases are also planned as later capabilities.

## Business Value

### Accelerate drug-discovery research

The main goal is to accelerate drug-discovery research by making
clinical evidence available to preclinical R&D functions. Today,
clinical trial data is primarily accessible for GxP analyses and work
closely tied to regulatory submissions. The platform is intended to
enable approved internal secondary use for broader R&D research
questions.

### Bring clinical and preclinical evidence together

Preclinical data is already available in a Databricks catalogue. Making
clinical datasets available through the same R&D data foundation creates
an opportunity to combine clinical and preclinical evidence and
investigate questions across stages of the drug-development process.

### Improve self-service access to data

By allowing scientists to discover and query clinical datasets from
RDDW, the platform should reduce barriers to finding and reusing data.
Researchers can use notebooks for their own analyses without requiring a
new application for every research question.

### Enable future research applications

A consistent catalogue and documented dataset structure provide a
foundation for future dashboards and applications based on agreed use
cases, including clinical protocol optimization and other research
questions.

## Release Plan

### Release 1 --- Accessible clinical data catalogue

Release 1 is successful when:

-   SDTM and ADaM datasets from ongoing and concluded studies can be
    transferred quarterly from Biowork or Domino Clinical.
-   Datasets are organized in a Unity Catalog catalogue according to the
    agreed compound, study, snapshot, and dataset model.
-   Basic ingestion checks and documentation are available.
-   The clinical catalogue is accessible from RDDW.
-   Onboarded R&D scientists can browse the catalogue and query datasets
    using Databricks notebooks.
-   The AI and Data Products team can operate the ingestion process and
    platform.

### Release 2 and beyond --- Cross-study research enablement

Potential follow-on capabilities include:

-   Automated ingestion.
-   A cross-study model for domains, endpoints, and entities.
-   Research-oriented grouping and discovery across studies.
-   Dashboards and applications for prioritized use cases.

These capabilities will be refined according to demonstrated business
value and agreed requirements.

## Ownership and Responsibilities

The **AI and Data Products team** will act as the platform team and will
be responsible for:

-   Quarterly ingestion.
-   Source-data correctness as a platform operational responsibility.
-   Access configuration.
-   Platform infrastructure and technical operations.
-   Basic ingestion checks and platform documentation.

**R&D scientists using RDDW** will be responsible for maintaining their
own analyses and notebooks.

The need for formal Data Owner roles remains open. Regardless of the
operational ownership model, legal approval and applicable
data-protection responsibilities must be established independently of
technical platform ownership.
