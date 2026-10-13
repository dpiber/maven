## Cel Deep Focus
Badam przepływ budowania modelu POM (profile / parent / CI-friendly `${revision}`) — od materializacji modelu do przejścia model → project — zaczynając od `impl/maven-impl/.../DefaultModelBuilder.java`, bo mapa wskazuje strefę 1 jako #1 aktywność + % fix, hub na każdej ścieżce runtime i ~88% co-change z legacy builderem.

## change-id
`model-builder-flow-analysis`

## Entry pointy (2–3)
1. `impl/maven-impl/src/main/java/org/apache/maven/impl/model/DefaultModelBuilder.java` — hub modelu M4
2. `impl/maven-core/src/main/java/org/apache/maven/project/DefaultProjectBuilder.java` — granica model → project / reactor
3. `compat/maven-model-builder/src/main/java/org/apache/maven/model/building/DefaultModelBuilder.java` — legacy twin (most M3↔M4)

## Pierwsze unknowns
1. Jak DI / ClassRealm / refleksja / SPI łączą wywołania poza tym, co widać w jdeps (mapa: unknown, nie „brak sprzężeń”).
2. Gdzie dokładnie drift mostu M3↔M4 między twinami `DefaultModelBuilder` (impl vs compat) — co musi iść w parze, a co już się rozeszło.
3. Jak duży jest blast ekosystemu pluginów poza reactor przy zmianie kontraktu modelu / SPI (japicmp / konsumenci zewnętrzni = unknown).
4. Czy edge case’y CI-friendly `${revision}` / consumer POM siedzą w pipeline model buildera, czy „wycieka” dalej do project buildera (mapa: reverty wokół CI-friendly).
5. Na ile wspólne IO / `.mdo` / codegen (`maven-support`) wymusza regenerację zamiast ręcznej edycji obu światów przy zmianie w tym przepływie.

## Co NIE wchodzi w zakres
1. Lifecycle / reactor / concurrent (`BuildPlanExecutor`, strefa 2) — osobny hub z cyklami pakietowymi.
2. `mvnup` i kampanie CLI oraz root `pom.xml` / Dependabot — fałszywe centra z mapy.
3. `its/core-it-suite` jako źródło prawdy — tylko lustro regresji po ustalonym przepływie, nie punkt startu researchu.
