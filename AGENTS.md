# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Critical Rules

**Don't modify spec/ files without explicit direction**
The spec/ directory contains the project's feature specifications. These are reference documents that define requirements and should only be changed when requirements themselves change.  The one exception is the spec/tmp/ directory, where all planning, research, and exploratory work files should be created.

**Don't create files outside the project directory**
Only create files within the project working directory, not in system directories like /tmp or in the user's home directory.

## Session Start

When starting a new session, review the following files:

* @spec/project-overview.md for project-specific details
* @spec/development-workflow.md for details on development workflow
* @spec/jobs.md for a list of jobs to be done
