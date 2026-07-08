# Repo-Type Playbooks

Read this after the Orientation stage. Match the repo to a type below, then follow that
playbook's exploration order and ask its type-specific questions. Detection signals are
hints, not proof — verify with the actual files. If a repo is a mix (e.g. a monorepo with
a frontend and a backend), name each part and run the dominant one's playbook first.

Every playbook still obeys the golden rule: label claims [fact] / [inference] / [unknown]
and cite evidence.

## Contents
1. Backend service / API
2. Microservices (multi-service repo or mesh)
3. Frontend / web app
4. Service-oriented monolith
5. Library / SDK
6. CLI tool
7. Data / ETL pipeline
8. ML system
9. Mobile app
10. Infrastructure-as-code
11. Embedded / firmware
12. Smart contracts
13. Notebook-heavy data science
14. Generic fallback (nothing fits cleanly)

---

## 1. Backend service / API
**Signals:** `main`/`server`/`app` entry, HTTP/gRPC framework deps, route or controller files,
a Dockerfile exposing a port, DB drivers/migrations.
**Explore in this order:** entry point → routing/handlers → service layer → data model & migrations →
external integrations (queues, other services, third parties) → auth → error/retry handling.
**Ask:** What are the main endpoints and who calls them? Where does a request enter and where does it
touch the database? What's the auth model? What external systems does it depend on, and what happens
when they fail?
**Artifacts to prioritize:** key request flow (`04`), data model in architecture (`02`), dependencies (`05`).

## 2. Microservices (multi-service repo or mesh)
**Signals:** many service dirs each with its own manifest/Dockerfile, service discovery/config,
gRPC/proto files or an event bus (Kafka/RabbitMQ/SNS-SQS), an API gateway.
**Explore in this order:** service inventory (name + one-line purpose each) → gateway/entry → how
services talk (sync RPC vs async events) → ownership boundaries → shared contracts (protos/schemas) →
failure & retry patterns.
**Ask:** How many services and what does each own? Synchronous or event-driven? Where are the contracts
defined? Which service is on the critical path for the main use case? What's the failure blast radius
when one service is down?
**Warn:** blast radius is subtle here — cross-service coupling via events is easy to miss. Flag it in `05`.

## 3. Frontend / web app
**Signals:** `package.json` with React/Vue/Svelte/Angular/Next, a `src/` with components/pages/routes,
a bundler config, a `public/`/`static/` dir.
**Explore in this order:** app entry & routing → page/route structure → state management (store/context/
query cache) → API/data layer (how it talks to the backend) → shared component library → build/deploy.
**Ask:** What are the main screens/routes? How is state managed and where does server data come from?
What's the component hierarchy for the primary user flow? How is it built and deployed?
**Artifacts:** route/screen map in `02`, the primary user flow in `04`.

## 4. Service-oriented monolith
**Signals:** one deployable, but internally organized into clear modules/packages by domain; a single
manifest; layered structure (controllers/services/repositories) or domain modules.
**Explore in this order:** module map → the domain each module owns → the layering convention →
one end-to-end flow across modules → shared/coupling points → what would be hard to change.
**Ask:** What are the domain modules and their responsibilities? What's the layering discipline? Where
do modules improperly reach into each other? Which module is the "center of gravity"?
**Artifacts:** module map (`02`), domain glossary (`03`), one cross-module flow (`04`).

## 5. Library / SDK
**Signals:** a public API surface, no server entry point, packaging/publish config, an `examples/` dir,
heavy README/API docs, semver.
**Explore in this order:** public API surface (exported functions/types) → core abstractions → typical
usage from README/examples/tests → internal architecture behind the API → extension points → versioning
& backward-compat rules.
**Ask:** What's the public contract vs. internal detail? What are the core abstractions a user works with?
What are the intended usage patterns (read the examples/tests)? What breaks compatibility?
**Artifacts:** API surface + core abstractions in `02`, usage patterns in `04`.

## 6. CLI tool
**Signals:** an arg-parser (argparse/cobra/clap/commander), a `bin/` or `cmd/` entry, subcommand files,
man/help text.
**Explore in this order:** command tree (commands & subcommands) → each command's job → shared core logic
they call → config/state/credentials handling → side effects (filesystem, network, external APIs).
**Ask:** What commands exist and what does each do? Where's the shared logic under the commands? What
does it read/write on the system? How is it configured and authenticated?
**Artifacts:** command tree in `02`, one representative command's flow in `04`.

## 7. Data / ETL pipeline
**Signals:** orchestration (Airflow/Dagster/Prefect/dbt), job/DAG definitions, warehouse/lake connectors,
transform SQL or Spark/pandas jobs, schedules.
**Explore in this order:** source systems → ingestion → transformations → destinations/sinks → scheduling
& orchestration → data quality/validation → failure handling & backfills.
**Ask:** Where does data come from and where does it land? What are the transform stages? What's the
schedule and what triggers runs? How are failures and reprocessing/backfills handled? Where's data
quality enforced?
**Warn:** correctness > structure here. Emphasize data lineage and what a bad run corrupts (`04`, `06`).

## 8. ML system
**Signals:** training scripts, model/experiment configs, notebooks, data loaders, `requirements`/`conda`
with ML libs, checkpoints/artifacts, serving/inference code, MLOps configs.
**Explore in this order:** problem & data → data pipeline/features → model & training loop → evaluation &
metrics → experiment tracking → serving/inference path → retraining/deployment.
**Ask:** What does the model predict and on what data? How is it trained and evaluated? How are
experiments tracked? How does a trained model reach production? How is it retrained/monitored for drift?
**Note:** distinguish research code (notebooks) from production code (serving) — they often have very
different quality and ownership. Call it out.

## 9. Mobile app
**Signals:** iOS (`.xcodeproj`/Swift/`Info.plist`) or Android (`build.gradle`/Kotlin/`AndroidManifest.xml`),
or cross-platform (Flutter `pubspec.yaml`, React Native), screen/navigation code.
**Explore in this order:** app entry & navigation → screen structure → state management → networking/data
layer → local persistence → platform integrations (push, permissions, native modules) → build/release.
**Ask:** What are the main screens and how does navigation work? How is state and local data handled?
How does it talk to the backend and cache offline? What native/platform features does it use? How is it
built and released to stores?
**Artifacts:** screen/navigation map in `02`, primary user flow in `04`.

## 10. Infrastructure-as-code
**Signals:** Terraform (`.tf`), CloudFormation, Pulumi, Kubernetes manifests/Helm charts, Ansible,
provider configs, environment dirs (`prod/`, `staging/`).
**Explore in this order:** what environments exist → what resources are provisioned → module/stack
structure → dependencies between resources → secrets/config management → the apply/deploy workflow →
blast radius of a change.
**Ask:** What environments and cloud resources are defined? How are modules/stacks organized? What
depends on what? How are secrets handled? What's the plan/apply workflow and its guardrails? What's the
scariest change to make here?
**Warn:** blast radius is highest of any type — a wrong apply can take down infra. Heavily emphasize `05`
and the "safe change + verify" guidance in `07`.

## 11. Embedded / firmware
**Signals:** C/C++/Rust with no OS assumptions, `Makefile`/`CMakeLists`/PlatformIO, linker scripts,
register/peripheral headers, `interrupt`/`ISR`/`HAL` code, RTOS (FreeRTOS/Zephyr), `.ld`/`.dts` files.
**Explore in this order:** target hardware & toolchain → boot/startup and `main` → memory layout (linker
script) → hardware abstraction / drivers → interrupt handlers & main loop or RTOS tasks → I/O and
protocols (UART/SPI/I2C) → build & flash workflow.
**Ask:** What chip/board and toolchain? What runs at boot and what's the main control loop or task set?
How is memory laid out? What peripherals/drivers exist? Where are the timing- or interrupt-critical paths?
**Warn:** correctness is timing- and hardware-dependent; much behavior can't be confirmed from source alone
— flag those as [unknown] and note they need hardware to verify.

## 12. Smart contracts
**Signals:** Solidity (`.sol`)/Vyper/Rust (Anchor/CosmWasm), Hardhat/Foundry/Truffle config, `contracts/`,
ABIs, deployment scripts, test suites in JS/TS or Solidity.
**Explore in this order:** contract inventory & inheritance → state variables and who can change them →
public/external entry functions → access control & modifiers → value flows (who can move funds) →
external calls & upgradeability → invariants the tests assert.
**Ask:** What contracts exist and how do they inherit? Who's authorized to do what? Where does value move?
What are the upgrade/admin powers? What invariants do the tests protect?
**Warn:** blast radius is financial and often irreversible; emphasize access control and value flows in `05`,
and be conservative — mark anything security-relevant you haven't fully verified as [inference]/[unknown].

## 13. Notebook-heavy data science
**Signals:** many `.ipynb` files, `data/` dirs, exploratory scripts, few or no tests, ad-hoc structure,
pinned analysis libs (pandas/numpy/sklearn/matplotlib).
**Explore in this order:** what question/analysis the repo answers → data sources & where they live →
notebook execution order (often undocumented — infer from naming/dates) → key transforms & outputs →
which parts are throwaway exploration vs. load-bearing → any productionized path.
**Ask:** What's the analysis trying to answer? What data feeds it and in what order do notebooks run?
Which notebooks produce the real outputs vs. scratch? Is anything here relied on downstream?
**Warn:** reproducibility and execution order are the main risks; call out hidden state and undocumented
run order as [unknown] rather than guessing a clean pipeline exists.

## 14. Generic fallback (nothing fits cleanly)
Use this when the repo doesn't clearly match a type above. Don't force a bad match — run the neutral
ladder and let evidence lead.
**Explore in this order:** what is this and what problem does it solve (README, docs, top-level layout) →
how it's organized (biggest/most-central dirs and files) → entry points or public surface (whatever the
outside world touches) → one important path traced end to end → what depends on what → how it's built/run.
**Ask:** What is this repo for? How is it structured and what's central? What's the "front door"? Pick one
representative thing it does and trace it. What would break if I changed the central piece? How do I build
and run it?
**Note:** classify as best you can and say so ("[inference] closest to a <type>"), then borrow the nearest
specific playbook's questions where they help.
