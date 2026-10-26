# Changelog

Release versions follow semantic versioning (MAJOR.MINOR.PATCH) and use Git tags with a `v` prefix.

## Unreleased

- Add the MIT License and README license badge while preserving existing file and package license notices.
- Correct the initial release author and tagger to Wonikrobotics-official while retaining contributor credit.
- Credit shy0914 as a contributor in the README and commit metadata.
- Use F and V consistently for model names in documentation and launch help.

## [1.0.0] - 2026-10-26

Initial GitHub release of the Allegro Hand V6 F software for Ubuntu 24.04 and ROS 2 Jazzy, based on the complete Jazzy source bundle dated 2026-10-22.

- Include the common libraries, V6 hardware driver, robot descriptions, controller bringup, and UI applications in one workspace.
- Document F-model installation and operation, with links to bringup and UI guides.
- Clarify that the F model does not require camera mapping; retain the shared F/V software and model assets.
- Include optional CSV recording with a configured 500 Hz control rate.
- Set package and Python distribution versions to 1.0.0.

The original software and model validation scope is recorded in [VALIDATION.md](VALIDATION.md). This release adds documentation and version metadata; physical hardware testing was not repeated.
