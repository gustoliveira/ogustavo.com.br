---
title: "Material and Flutter: A New Standard and an Old Debate in the Community"
date: 2025-05-16
draft: false
description: "Explore the ongoing debate about Material Design's coupling with Flutter, challenges with Material 3 Expressive, and the community's perspective on framework dependencies."
---

This Tuesday, the new thematic standard for Android was officially revealed: Material 3 Expressive, aiming to bring a more modern system with many animations.

However, not only is it not yet available, but we also don't have a date for when this new package will begin development for Flutter. According to the team's statement, although this new version was known internally, being an open-source framework meant development couldn't commence.

Consequently, this new announcement has reignited a constantly debated issue within the community: **the coupling of Material Design with the Flutter framework**.

This was a deliberate choice by the team, with the primary goals of:

- Providing a simpler and more organic out-of-the-box experience, eliminating the need to add or study separate packages to build UIs;

- Ensuring consistent and optimized implementation, as the core team would maintain the complete visual package.

However, this has created new challenges:

- Fixes for visual inconsistencies are delayed, as they only become available with new Flutter releases;

- The necessity of including Material Design even when not strictly used, increasing the application's bundle size;

- Difficulty in migrating versions, as developers are forced to upgrade the entire framework to access the new package;

- Having the framework's development roadmap tied to Material Design.

Indeed, these headaches aren't new. In 2023, with the release of Flutter 3.16, Material 3 became the default in the framework. This led to complaints, such as Material 3 arriving incomplete and many changes not being clearly explained in the documentation or migration guide.

These issues have been extensively discussed in GitHub issues, and the core team has raised some understandable points, particularly regarding the difficulty: transforming Material Design into a separate package would be a complex, arduous, and lengthy task, and many question its overall impact.

In my opinion, it would be a worthwhile investment. Once fully separated, we would have:

- Separate roadmaps and release schedules for the framework and Material Design;

- More frequent releases with new features and patches for Material Design;

- Reduced workload for the Flutter team, as unrelated issues could be prioritized differently.

Ultimately, when dealing with a framework as comprehensive as Flutter, decisions like these are complex and require extensive planning. However, I believe separating them would allow for faster development on both fronts, which is crucial for the tool's success.

For now, what we have (or rather, don't have, lol) is a published plan detailing how and when this new version will begin development, the methods to be used, and the lessons learned from previous migrations.

### References

- BROOKS, M. Android and Wear OS are getting a big refresh. **[Available here](https://blog.google/products/android/material-3-expressive-android-wearos-launch)**. Acessed 13 may. 2025.

- Expressive Design: Google's UX Research. **[Available here](https://design.google/library/expressive-material-design-google-research)**. Acessed 15 may. 2025.

- Github Issue: Move the material and cupertino packages outside of Flutter #101479. **[Available here](https://github.com/flutter/flutter/issues/101479)**. Acessed 15 may. 2025.

- Github Issue: ☂️ Bring Material 3 Expressive to Flutter #168813. **[Available here](https://github.com/flutter/flutter/issues/168813)**. Acessed 15 may. 2025.
