# **Level 2**

The level two architecture is built on top of level one; it allows
communities to run benchmarking experiments at OpenEBench by using
benchmarking workflows to assess participants' performance. Those
workflows compute one or more evaluation metrics given one or more
reference datasets, and produce several assessment and aggregation
datasets (see section Benchmarking Data Model: Data Types and
Cross-References), which are compatible with Level 1, so they can be
easily exported to OpenEBench' Mongo database for long-term storage.

## Workflows structure

As seen in the Figure below, our benchmarking workflows are composed of
three conceptual blocks (that might be formed of one or more workflow
steps), which we encourage the communities to follow for compatibility
with our system. Those blocks are:

1.  **Validation and preprocessing**: the input file format is checked and, if required, the content of the file is validated (e.g check whether the submitted file contains certain fields, or compare to a 'public reference' dataset to check if the submitted IDs exist). These are the parameters involved in this step (for more information about parameters visit the 'workflow parameters' structure):

-   Inputs: input, community_id, challenges_ids, participant_id, public_ref_dir

-   Outputs: validation_result (and an exit status which indicates whether the validation was successful or not).

2.  **Metrics Computation**: the predictions are compared with the \'Gold Standards\' provided by the community, which results in one or more performance metrics (e.g. Precision & Recall). These are the parameters involved in this step (for more information about parameters visit the 'workflow parameters' structure):

-   Inputs: input (depending on whether the validation step was successful), community_id, challenges_ids, participant_id, goldstandard_dir.

-   Outputs: assessment_results.

3.  **Results Consolidation**: the benchmark itself is performed by merging the tool metrics with the rest of the community' reference data. The results are provided in JSON format and SVG format (scatter plot). These are the parameters involved in this step (for more information about parameters visit the 'workflow parameters' structure):

-   Inputs: assessment_results, validation_result, assess_dir

-   Outputs: outdir, data_model_export_dir

As seen in the Figure's green box, all three steps produce a series of
Datasets which are compatible with the [[Elixir Benchmarking Data
Model]{.ul}][13] (see section).

![][32]

For reproducibility and interoperability purposes, OpenEBench encourages
communities managers to submit their pipelines wrapped with a workflow
management system (e.g [[Nextflow]{.ul}][33]) and its tools
containerized (e.g. [[Docker]{.ul}][34]). **NOTE for developers, in
order to make the workflow containers reproducible and stable in the
long-term, make sure to use specific versions in the container base
image (e.g.ubuntu:16.04, NOT ubuntu:latest).**

For more information about how to build your own benchmarking workflow,
see our TCGA sample workflow at
[[https://github.com/inab/TCGA_benchmarking_workflow]{.ul}][35]. **NOTE
for developers: In order to make your workflow compatible with the
OpenEBench infrastructure, please make sure to use the same 3-step
structure, output formats, and parameter names in it.**

#### Scientific Benchmarking: workflow parameters

Description of the parameters used in OEB benchmarking workflows:

-   **[INPUTS]{.ul}**

    -   **input**: predictions file submitted by the participants

    -   **public_ref_dir**: directory which contains one or more
        > reference files used to validate input data.

    -   **participant_id**: name of the tool used for the predictions.

    -   **goldstandard_dir**: directory where the \'gold standards\' or
        > 'reference data' to compute the metrics are found.

    -   **challenges_ids**: list of challenges (performance evaluation
        > methods) which are performed in the benchmark - if you have
        > only one evaluation method, just define a name for it.

    -   **assess_dir**: directory where the performance metrics for
        > other participants to be compared with the submitted one are
        > found. If there is no other benchmark data yet, an empty
        > aggregation dataset should be defined.

    -   **community_id**: name or OEB permanent ID for the benchmarking
        > community.

-   **[OUTPUTS]{.ul}**

    -   **validation_result**: file path where it is written the
        > validated participant JSON, which corresponds to a minimal
        > dataset compatible with the [[Elixir Benchmarking Data
        > Model]{.ul}][13] (see section).

    -   **assessment_results**: file path where it is written the set of
        > assessment datasets in JSON, which corresponds to minimal
        > datasets compatible with the [[Elixir Benchmarking Data
        > Model]{.ul}][13] (see section).

    -   **outdir**: directory where the run results are saved - one or
        > more aggregation files used by the visualization, and several
        > SVG/PNG plots.

    -   **statsdir**: directory where all nextflow statistics (timeline,
        > trace, report...) are written.

    -   **data_model_export_dir**: file where all the datasets generated
        > during the workflow, which are compatible with the [[Elixir
        > Benchmarking Data Model]{.ul}][13] (see section), are merged
        > into a single JSON, which is ready to be validated and pushed
        > to Level 1.

    -   **otherdir**: directory where other community' specific results
        > can be written.

**NOTE for developers: In order to make your workflow compatible with
the OpenEBench infrastructure, please make sure to use the same
parameter names in it.**

#### Scientific benchmarking: Virtual Research Environment (VRE)

The [[OpenEBench Virtual Research Environment (VRE)]{.ul}][36] is the
e-infrastructure where OpenEBench Level 2 is underpinned. It integrates
OpenEBench resources with the purpose of developing, evaluating, and
offering unified benchmarking workflows useful for the different
scientific benchmarking paradigms.

OpenEBench VRE offers a complete web interface which brings together
public and/or consolidated benchmarking datasets, private participants
data, and the necessary mechanisms to import and execute benchmarking
workflows on top of a cloud computing infrastructure, for instance, the
ones at the [[Barcelona Supercomputing Center (BSC)]{.ul}][37]
facilities.

The platform serve different purposes to different users:

-   **Scientific communities** provide reference datasets and bioinformatic methods metrics.

-   **Software developers** evaluate their tools against community defined datasets and scientific challenge events, and compare their performance against other methods in the same field.

-   **Researchers** can objectively compare bioinformatic methods through the community defined metrics, in order to choose the most effective software for his research purposes.

Benchmarking workflows consist of three steps (visit 'workflows
structure' section for more information on this):

-   Validation of participants result.

-   Computation of the community-agreed metrics over such results.

-   Metrics consolidation obtained by combining and comparing all challenge participants metrics.

Implemented as three conceptual steps using software containers, the
community manager is able to import and compose the full benchmarking
workflow at OpenEBench VRE, where a scalable and virtualized environment
is settled for the workflow run, internally orchestrated by, at the
moment, the Nextflow workflow manager. Also, the community responsibles
can provide customized visualization methods to browse participant
results, individual metrics and/or assessment.

#### Scientific benchmarking: Community Manager Role

In order to integrate a workflow in the Virtual Research Environment,
managers associated to scientific communities have to first define
reference datasets and metrics to be used in a benchmarking workflow.

Once the metrics and reference datasets are defined, the workflow steps
need to be set, preferably using Docker containers and a workflow
manager such as Nextflow (visit 'workflows structure' section for more
information on this).

If OpenEBench guidelines and good practices are followed, the workflow
should be ready to be integrated at the platform in the frame of a
certain benchmarking event for that community. In order to do that, the
following steps need to be performed:

1.  Workflow should be publically available in a Git repository, a URL and specific commit hash need to be provided.

2.  Docker images have to be built in the VRE backend, which can be done by either providing OpenEBench team the required Dockerfiles, or uploading them as public containers to [[Docker Hub]{.ul}][38].

3.  Create a new entry in VRE Tools database, specifying the workflow, reference data, inputs & output parameters to be used, and their associated VRE metadata.

4.  Make that entry available in the VRE interface as a new benchmarking workflow, so that software developers can test their methods at the workspace.

Also, it is highly recommended to fill in the [[VRE Help]{.ul}][39]
section of the Tool/Workflow to inform users about how to test their
methods (e.g. formats, parameters...).

![][40]

#### Scientific benchmarking: Software Developer Role

Software developers are the end users of Level 2 benchmarking workflows.
They upload to the platform the results of the method interested in
evaluating (i.e. list of candidate genes, predicted 3D structures,
modeled phylogenetic tree).

By selecting the relevant benchmarking workflow, the metrics qualifying
the given data are computed. A graphic visualization (see 'visualization
and interpretation of results' section) is offered to comparatively
analyse the obtained metrics with other participating method metrics.

If results are satisfactory, the developer can proceed to request for
their publication - following community directives -, making them
available at Level 1 in the long-term for all OpenEBench users. If not,
they can also rerun the workflow with new data, and compare the results
against themselves until they are satisfied with their performance.

Please, refer to VRE *'Help'* section to read the instructions on how to
[[Upload Data]{.ul}][41] or [[Launch a Workflow]{.ul}][42].

![][43]

#### Scientific benchmarking: VRE Tool Specification Summary

[[https://docs.google.com/document/d/1Fid4RkNyt9-\_0g_SrCw8J1k8MrOMZI4GLzzpdCAATZc/edit\#]{.ul}][44]

#### Scientific benchmarking: What's behind Virtual Research Environment?

The VRE is conceived as a PaaS (platform as a service) where community
managers populate the infrastructure with benchmarking workflows. These
are formalized as a nextflow pipelines (fetched from GIT repositories)
where each step corresponds to a docker container (locally built or
published at Biocontainers).

VRE deploys the benchmarking environment as virtual machines on top of a
cloud infrastructure, either on-demand and in a distributed manner if
PMES enactment service is enabled - thanks to OCCI standards -, or
locally if the SGE batch queue system is used.

**VRE Main Components**

*Cloud deployments:* VRE infrastructure has been designed as a fully
virtualized environment. At its present state, VRE has been deployed in
at the Starlife cloud infrastructure
(https://www.bsc.es/marenostrum/star-life), at the Barcelona
Supercomputing Center, using OpenNebula (https://opennebula.org/) and
the KVM hypervisor (https://www.linux-kvm.org).

*Process management:* VRE uses two complementary layouts for process
management: i) Sun Grid Engine (SGE,
[[https://sourceforge.net/projects/gridscheduler/]{.ul}][45]), in
combination with OneFlow
([[https://docs.opennebula.org/5.4/advanced_components/application_flow_and_auto-scaling/appflow_use_cli.html]{.ul}][46]),
a component of the OpenNebula framework that allows managing Multi-VM
applications and auto-scaling. SGE is used to manage applications where
no complex workflows are necessary, requiring only to deploy additional
workers on peaks of demand, ii) the COMPS Superscalar (COMPSs)
programming model (and its python binding pyCOMPSs (39)), managed by the
Programming Model Enactment Service (PMES) (40), which interacts with
cloud infrastructures through Open Cloud Computing Interface (OCCI,
<http://occi-wg.org/>) servers. PMES/pyCOMPSs are used to control
complex workflows and distributed execution.

*Database manager:* Operational data and metadata regarding installed
tools, public repositories, and user's files are maintained in a MongoDB
database ([[https://www.mongodb.com]{.ul}][47]). User's data is stored
in a standard filesystem in its original format.

*Authentication and authorization system:* Data privacy is maintained
using the authentication and authorization server Keycloak
([[http://www.keycloak.org/]{.ul}][48]) to handle all internal
communications and user access. Keycloak implements OpenID Connect which
allows for Web access on the code authorization flow of OAuth2, and a
token-based authentication for the REST services. For registered users,
authentication schemes based on username/password, but also third-party
identity providers (Google, LinkedIn, Elixir) are accepted.

#### Scientific Benchmarking: VRE Workspace

**User access**

OpenEBench VRE can be accessed without authentication. Users are granted
a private workspace to hold data and analysis results. Data is
maintained for one week after the last access and can be recovered
during this period through a unique URL address. Users desiring a longer
interaction with OpenEBench VRE are advised to register to get a
permanent workspace. In such cases the user space is maintained up to
two months after the last access.

**Personal workspace**

The VRE personal workspace is the central environment for user activity.
It is based on a filesystem-based layout, where both uploaded data and
analysis results are available. Uploaded data should be annotated to
specify data types and formats. This allows the workspace to offer an
adapted toolkit for each file, including only compatible tools and
visualizers. The user workspace has been laid out to provide an
intuitive look-and-feel. The workspace itself is structured in projects,
to keep data logically organized. Within each project the input data is
distributed in folders: *Uploads* (uploaded data), and *Repository*
(data obtained from public repositories). The remaining folders
correspond to analysis operations (a new folder is generated for any new
process started in the VRE). File lists can be filtered by any of the
fields (name, format, data type, or project). Additionally, a tool-based
filter is available to select only valid input files for the given tool.

Three interactive toolkits containing the following options are
available:

-   File toolkit: Download, rename, move, compress, delete files and folders, edit their metadata, rerun jobs.

-   Visualization toolkit: Available visualizers for the specific file/s selection (based on their data type and format).

-   Tools toolkit: Available tools for the specific file/s selection (based on their data type and format).