# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.6] - 2026-06-08

- Update solana-verify to v0.5.0

## [0.3.5] - 2026-05-15

- Update all github-actions to v0.2.10

## [0.3.1] - 2026-04-29

- Update all github-actions to v0.2.9
- Update squads-program-action to v0.4.0
- Add program-metadata IDL upload support via `use-program-metadata-idl` input (default: true)
- Integrate `metadata-upload` and `write-metadata-buffer` actions
- Support both direct metadata upload and Squads multisig buffer workflow
- Expose `metadata_buffer` output from reusable workflow

## [0.2.11] - 2024-07-26

- Update github actions to use v0.2.7

## [0.2.10] - 2024-07-26

- Pass library-name to export-pda-tx

## [0.2.7] - 2025-05-02

- Make program-id input mandatory and add a check that program keypair secret and program-id input match

## [0.2.6] - 2025-04-25

- Updated Node.js requirement to >=20.11.1 for compatibility with @solana/codecs-numbers

### Changed

## [0.2.4] - 2024-03-21

### Changed

- Added delay before verify build step to ensure program upgrade is finalized before verify

## [0.2.2] - 2024-03-21

### Changed

- Fix issue in extend program action

## [0.2.1] - 2024-03-21

### Changed

- Fix issue in extend program action

## [0.2.0] - 2025-02-05

### Changed

- Updated CI/CD pipeline with reusable workflow configurations
- Enhanced build and test automation
- Now using solana actions version 0.2.0
- Updated readme
