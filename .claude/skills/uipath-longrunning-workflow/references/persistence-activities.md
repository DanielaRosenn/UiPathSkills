# Persistence Activities Reference

This document provides a reference for the various persistence activities available in UiPath Long Running Workflows.

## Activity Overview

### 1. Save Workflow State
- **Description:** Saves the current state of the workflow to a specified storage.
- **Inputs:**
  - State Name
  - Storage Location
- **Outputs:**
  - Confirmation Message

### 2. Load Workflow State
- **Description:** Loads the previously saved state of the workflow.
- **Inputs:**
  - State Name
  - Storage Location
- **Outputs:**
  - Workflow State Object

### 3. Delete Workflow State
- **Description:** Deletes a previously saved state from the storage.
- **Inputs:**
  - State Name
  - Storage Location
- **Outputs:**
  - Confirmation Message

## Best Practices
- Always validate the state information before saving or loading.
- Implement error handling to manage possible failures during these operations.