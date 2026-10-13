---
change_id: model-builder-flow-analysis
title: Analiza przepływu budowania modelu POM (M3↔M4)
status: preparing
created: 2026-10-13
updated: 2026-10-13
archived_at: null
---

## Notes

Badam przepływ budowania modelu POM (profile / parent / CI-friendly `${revision}`) — od materializacji modelu do
przejścia model → project — zaczynając od `impl/maven-impl/.../DefaultModelBuilder.java`, bo mapa wskazuje strefę 1 jako
#1 aktywność + % fix, hub na każdej ścieżce runtime i ~88% co-change z legacy builderem.
