# Contract: relativePath GA mismatch (impl ↔ compat twin)

**Verdict:** intentional limitation (MNG-5146 / `maven.maven3Personality`), not a bug. Do not unify severity.

## Severity matrix

| Side | Condition | Severity |
|------|-----------|----------|
| impl (default) | explicit `<relativePath>` GA mismatch | **FATAL** |
| impl (personality) | same mismatch + `Constants.MAVEN_MAVEN3_PERSONALITY` / `Features.mavenMaven3Personality` | **WARNING** |
| compat | same mismatch (always) | **WARNING** |

## Anchors

- Personality flag: [`Constants.MAVEN_MAVEN3_PERSONALITY`](../../../api/maven-api-core/src/main/java/org/apache/maven/api/Constants.java) / [`Features.mavenMaven3Personality`](../../../api/maven-api-core/src/main/java/org/apache/maven/api/feature/Features.java)
- Impl hot path: [`DefaultModelBuilder.mismatchRelativePathAndGA`](../../../impl/maven-impl/src/main/java/org/apache/maven/impl/model/DefaultModelBuilder.java) (`warn = defaultPath \|\| maven3Mode`)
- Compat WARNING path: [`compat … DefaultModelBuilder`](../../../compat/maven-model-builder/src/main/java/org/apache/maven/model/building/DefaultModelBuilder.java) (~1072–1074)
- Existing FATAL unit: [`ParentCycleDetectionTest`](../../../impl/maven-impl/src/test/java/org/apache/maven/impl/model/ParentCycleDetectionTest.java) (`testWrongRelativePathInGeneratedPomIsFatalWithMaven3Hint`)
- Compat MNG-5146 WARNING: [`DefaultModelBuilderTest`](../../../compat/maven-model-builder/src/test/java/org/apache/maven/model/building/DefaultModelBuilderTest.java)
- Research: [research.md](./research.md) (K2 — świadome ograniczenie)

## Gap (phases 2–3)

Impl has FATAL coverage; **no** unit yet for personality demotion → WARNING. Phases 2–3 close that characterization gap (and paired review vs MNG-5146) without syncing twin severity.
