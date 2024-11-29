## Software Assurance in Critical Systems
Critical systems are safety-focused, highly available, and provide a long service life. These traits align nicely with Bitcoin. Software assurance is the practice of employing software development processes to produce more reliable, stable, secure, and functional software to be used in essential applications. The following discusses common software assurance practices used for critical systems development, and potential future areas of discussion for Bitcoin Core. The practices discussed are a sampling from industry standards rather than an exhaustive list.

Process that works well for one environment doesn't necessarily work well in another. The intent is to provide general awareness rather than to modify or apply new processes to Core.

### Alignment to Standards
Software Assurance standards provide a systematic, organized framework for software development process.

Some standards focus on prescriptive measures (i.e. conducting process activities/tasks is assumed to result in safer software). Others are outcome-focused (i.e. ensure the objectives are met, with the enabling processes being necessarily sufficient). Some standards allow for a selection or a blend of prescriptive or outcome-focused process.

**Example standards**
- ISO/IEC 12207 (Systems and Software Engineering - Software Life Cycle Processes)
  - General guidance for software life cycle processes, not specific to a particular industry
  - Permits either conformance to tasks (required activities of the declared set of processes are achieved) or outcomes (required objectives of the declared set of processes are achieved). Conformance of one kind relegates the other to guidance.
- DO-178C (Software Considerations in Airborne Systems and Equipment Certification) and DO-278A (Software Integrity Assurance Considerations for Communication, Navigation, Surveillance and Air Traffic Management Systems)
  - Standards specific to the aviation industry, both for airborne equipment (e.g. planes) and for the ground systems responsible for providing reliable communication and supporting capabilities to air traffic controllers
  - Allows for variation in process based on designated assurance levels, allowing designation of more critical and less critical software

#### Formalized Process
Although these standards do allow for some degree of ahead-of-time tailoring, the chosen/declared process is considered mandatory. This can be a tradeoff. The quality of the produced software is generally more consistent with adherence to process. However, a lack of on-the-spot flexibility can lead to decay of intent and focus on "following the rules" rather than doing what makes the most sense in a given situation. Compliance for compliance's sake can cause inefficient use of available resources. Balance between too little or too much process can be an evolution for a project.

#### Requirements-Focused
As the typical perspectives and roles of these standards is that of the stakeholder, acquirer, and supplier, focus is heavily placed on establishing formal requirements to drive development. Stakeholder communication is established and maintained throughout the development process. Stakeholders witness and audit development activities. Stakeholder needs are memorialized in high-level system requirements. Software requirements are created from the high-level system requirements. The requirements cover functional and performance needs comprehensively.

Requirements are both decomposed (creating one or more detailed requirements from a high-level requirement) and derived (requirements introduced by way of inference from the decomposition process). The requirements themselves are validated for consistency and alignment. For example, a decomposed requirement cannot conflict with its parent requirement. It is ensured that software is developed from, and is mapped to, each requirement. No requirements have missing implementation and no implementation is created without a driving requirement. The aim is to prevent the development of incomplete or unnecessary software. One can imagine that this may also have the downside of disincentivizing the creativity that may lead to new innovation, or at minimum, leaves this exploration to outside efforts.

Interface-focused requirements are defined to ensure communication between components is not only functional, but clear to each implementing party, as well as maintainable as changes are made to either side of the interface.

All requirements developed must be verifiable, i.e. testable.

All requirements are formally documented and change-controlled.

#### Risk Management
Risk management process is included in the overall software process of standards, which requires identifying, analyzing, prioritizing, treating, and tracking risks. Tools such as risk matrices are used to weight risk based on both severity and likelihood. Risk always exists, and management of risk provides the opportunity to proactively guide toward desired and less costly outcomes. Conducting risk management in a more formalized process can also increase confidence, reducing paralysis and rumination.

#### Structure and Architecture
Standards require organizing and establishing the overall architecture of the system in terms of elements and components. Software architecture is developed from the system and software requirements. The architecture is validated to be consistent and compatible with the requirements.

Strong emphasis is placed on partitioning and fault separation, such that failure of one component of the architecture does not cause direct and catastrophic failure of another component. Failure events are planned for and included in the design. Redundant components are used to prevent a single point of failure. Seamless failover algorithms are employed such that both failure and recovery behavior are known ahead of time. System elements and components are designed with maintainability in mind, such that these components can be removed and replaced within a defined and constrained timeframe (e.g. Mean Time to Repair).

##### Multiple-Version Dissimilar Software
Purposefully different implementations of the same functionality can be used to incorporate diverse redundancy and avoid common failure conditions. This is typically reserved for the most critical applications, in addition to other objectives being met.

##### Partitioning
Partitioning provides isolation between software components to prevent cascading failure conditions. This can be achieved through separate hardware (failure of one "box" doesn't bring down another) as well as with internal software boundaries (e.g. virtualization, containerization, IPC, etc.). Care is taken to prevent failure impacts that are more nuanced (e.g. atomic operations, shared resource exhaustion, contamination of an adjacent partition's storage, I/O, or code). Even the mechanisms providing and facilitating the partitioning are assessed to ensure they are of a similar or more rigorous assurance level as the software being partitioned.

#### Hazard Analysis and Assurance Level
Systems undergo a hazard analysis, which is a risk management activity. A hazard analysis identifies the loss or harm associated with adverse events. In the DO-278A standard, hazard analysis guides the designation of software components as more or less critical, allowing the components to be bucketed into a defined assurance level. The assurance level has corresponding objectives and activities, with more critical assurance levels associated with additional and more rigorous objectives, activities, and oversight/auditing. In this way, resources are prioritized toward more critical components of the system.

As an example for aviation-related systems, within DO-278A the following assurance levels are defined:

| Assurance Level | Failure Condition |
|-----------------|-----------|
| 1               | Catastrophic |
| 2               | Hazardous |
| 3               | Major |
| 4               | None* |
| 5               | Minor |
| 6               | No effect |

*No failure condition is associated with AL4 as a result of a mapping between DO-278A and DO-178C. AL4 is more rigorous than AL6.

Although DO-278A focuses on categories based on safety impact, one could consider an alternative where financial impact or key operational failures (e.g. consensus failure) are used for level determination.

| Category     | Description |
|--------------|-------------|
| Catestrophic | Multiple fatalities |
| Hazardous    | Serious or fatal injury |
| Major        | Multiple non-serious injuries |
| Minor        | Physical discomfort |
| No effect    | No effect |

As an example, software designated AL1 or 2 must have independent entities verify the accuracy and consistency of requirements, algorithms, compatibility of the software architecture with the requirements, integrity of partitioning, database correctness, and the correctness of test procedures/results/discrepancies/coverage. AL3 and above do not require an independent entity to perform these activities.

#### Usage of COTS
Usage of Commercial Off the Shelf solutions can reduce development burden, but comes with the tradeoff of having less control of the development process for creation and maintenance of the COTS solutions. COTS comes in different forms. Dependencies developed outside of the system (e.g. external libraries) could be interpreted as COTS.

Several factors are included when assessing the suitability of COTS for incorporation into the system. Often, a record of significant service history of particular instances of the COTS is considered for assurance of stability.
- Service duration length (how long the instance of the COTS implementation has been operating in a similar/suitable environment)
- Service history data quality and availability
- Change control during service (assurance that changes to the instance were known and managed)
- Proposed use versus service use (how the instance was used compared to how it aims to be used by the system)
- Number of significant modifications during service (the number of times the instance underwent planned modification/update)
- Error detection and reporting capability (assurance that errors would be known)
- Number, severity, and impact of in-service errors encountered

#### Verification
Verification activities are a major part of critical system development, and typically exceed greater than half of the development cost of a critical system. Critical systems are not deployed without having undergone formal verification to ensure each requirement is satisfied and all discrepancies have been reviewed and accepted by stakeholders.

##### Independence
Standards require independence of the verification activities, i.e. that testing is performed by individuals other than the developer of the software. This is done to prevent conflict of interest (e.g. the developer may be incentivized to downplay anomalies or omit findings in order for success to be declared). It is not uncommon for independent organizations to be established for this purpose.

##### Test Environment
Verification typically can only achieve at best close approximations of the actual operating environment in which the software will be used. Standards require analysis, rationale, and justification for significant deviations from a realistic test environment.

Strong configuration management is required during test activities. The environment in which test activities are run must be reproducible, ensuring testing is vetting the operational/representative configuration, and aiding in anomaly recreation during investigation.

##### Traceability and Verification Method
All requirements of the system are verified or otherwise waived, and this traceability is maintained. Requirements can be verified using one or more methods, e.g.: Inspection, Analysis, Demonstration, Test. Flexibility is granted to allow a requirement to be verified using the most appropriate verification method(s), but employing no verification method is unacceptable.

For the Test method, testing is accomplished at multiple levels: Unit (specific to software components), Integration (between components), System (the system as a whole), Inter-System, Production (typically hardware-focused), Deployment (deployed system is tested prior to first operation).

##### Tool Accreditation
All tools used for testing undergo their own verification to ensure they are accurate and functional. This reduces the risk that a tool provides a false failure or false pass indication during testing, increasing confidence in the test activities and results.

##### Interface Testing
Interfaces are tested alone (with stubs/drivers), and through integration test activities with other components of the system (as well as components of other systems).

##### Anomaly Review
All anomalies/bugs are reported, reviewed by multiple parties (including stakeholders), and are dispositioned appropriately.

##### Performance Testing
Verification of performance requirements typically involves the use of highly specialized tools to accurately measure component performance characteristics without introducing significant observer effect.

##### Stability Testing
Testing the system under operational conditions for an extended duration under heavy monitoring is important for the detection of issues that are intermittent or slow to appear. It is not uncommon for weeks to months of stability testing to be conducted before considering the system "stable enough" for operational use.

##### Coverage
Software branch/conditional coverage is assessed, to ensure test coverage is as comprehensive as possible.

#### Other Practices

##### Documentation
Throughout the development process, comprehensive documentation is created and maintained. This includes system documentation (e.g. for training, manuals, and design information), verification artifacts, and documentation of the formal processes and plans themselves. Documentation development and maintenance introduce a significant effort, promote developer succession, and enable successful development process audit. Accurate documentation is also key for Knowledge Management, which includes ensuring practice and approach are shared across the organization and with onboarding team members.

##### Quality Assurance
Independent quality assessors audit the level to which the processes are being followed. Non-conformance to the processes signals program risk and misalignment with agreements.

#### Training, Transition, and Deployment
Training materials are generated to ensure users are able to operate the system effectively. Transition constraints are identified to ensure continuous service availability during the transition from an existing implementation to a new one. Planning is done to optimize deployment strategy. It is not uncommon for deployments to take months to years to complete.

### Bitcoin Core Development Process
The following touches on a few areas of Bitcoin Core development process.

#### Requirements
The contributors to Bitcoin do not aim to have a comprehensive set of formal requirements. BIPs are created to discuss and formalize proposed improvements to Bitcoin or to provide general clarity to implementors. Bitcoin Core, being the reference implementation and by far the most widely used implementation, serves as the defacto specification (i.e. the code is the spec).

Requirements and implementation of those requirements can deviate initially or over time (and the goal of testing is to discover this deviation). In a typical system, it wouldn't necessarily be controversial to update the implementation to align with requirements specification. In an application where backward compatibility is a necessity and there is no central control over the versions of software being run by node operators, adjusting the implementation to align with requirements can cause consensus failure or undesired splits in the network. Because consensus is so critical to the health of the Bitcoin network, over time nuances of the implementation have essentially become part of the specification of the protocol (e.g. the CHECKMULTISIG bug).

#### Documentation and Knowledge Management
[`doc/`](https://github.com/bitcoin/bitcoin/tree/master/doc) contains user, design, and process documentation. [`doc/developer-notes.md`](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md) contains useful guidance on preferred approaches, including coding style, architectural approaches/practices, and nuances/tribal-knowledge. Other resources outside of the Bitcoin Core repository exist as well (e.g. https://bitcoincore.academy, notes from past/current Core developers, etc.).

#### Testing
Test creation is at the author's discretion, but insufficient testing will likely delay or prevent the merging of proposed changes. Reviewers typically recognize gaps in test coverage and recommend additional test creation. Code coverage tooling also enables the identification of unexercised code, for which tests can be created to increase coverage.

Core includes significant automated test capabilities, including unit tests, functional tests (which act as a blend of integration and system-level tests), benchmarks, and fuzz tests. Linting is used to maintain code style and catch common programming errors.

Continuous Integration, using multiple platform types, provides a means to detect test failure in a consistent fashion. Review of failure is done by the author and reviewers.

##### Review
Pull request review provides a human element to verification of changes (e.g. through code review and targeted exercising/testing). Maintainers merge pull requests only after significant review and testing have been performed. While there is not a firm metric to meet, the scope and impact of the change as well as the quality of the review is considered.

##### Test Networks
Testnet and Signet provide a means to test changes in an environment mimicking the Bitcoin network, albeit at a much smaller scale. Regtest is typically used for local testing in a controlled and repeatable fashion.

### Potential Areas of Further Discussion
Given the discussion of software assurance standards and the Core development process, the following may be pertinent areas for further discussion.
- Stability testing
  - While there are open Bitcoin test networks (signet, testnet), it is unclear the extent to which nodes on these networks are monitored. Long-term monitoring can uncover instabilities (e.g. slow or accumulating faults) and other bugs/issues. Some tools are being created today (e.g. Warnet), which could potentially help with stability testing.
- Increased testing with stakeholders
  - Although Release Candidate builds of Core are available prior to final release, occasionally downstream users present complaints about a release breaking a particular use case. This may be a symptom of limited stakeholder testing prior to release. It could be worthwhile to attempt increased stakeholder engagement to enhance testing culture, catch issues prior to final release, and help users be better prepared for changes.
- Assurance level designation
  - Assurance level designation for portions of critical systems allows for setting a baseline/bar for the level of rigor that needs to be employed before a change to the system is considered. There is no recipe for the adoption of changes in Bitcoin, and one is not anticipated to arrive soon. While introducing something analogous to assurance level doesn't guarantee a change will incorporated, it could provide some additional confidence that due diligence has been followed, and encourages a clearer path for outsiders wishing to make what they consider to be valuable changes to Bitcoin.
- Failure Mode and Effects Analysis
  - Currently, it is up to reviewers to spot how a change can fail or cause failure to an associated component. Typically the author provides tests (unit, functional, benchmark, fuzz, etc.), but it might be beneficial to provide additional guidance to authors on which types of failure modes should be considered and explicitly shared with reviewers.
- Monitoring of dependencies
  - An overall trend appears to be a reduction of dependencies (e.g. removal of BDB). Removal of dependencies is a tradeoff (typically more code to maintain internally)
  - Some dependencies may become less/unmaintained over time, or contain/introduce vulnerabilities. Active monitoring of the state of dependencies can help Bitcoin Core stay responsive to issues that may arise.
  - Guidance on service history of incorporated dependencies can help prevent false confidence in the quality of the dependency (i.e. "Has it really been used in a similar environment, or for this purpose?")

