# Grade student laboratory

## 1. Get the rubric

It's necessary to have a rubric associated with the laboratory in order to grade it:

```mermaid
sequenceDiagram
    Teacher -) Web: Select student to grade laboratory
    Web -) Main API: Get laboratory rubric

    alt There is no rubric
        activate Main API
        Main API --) Web: Rubric not found
        deactivate Main API
    else There is a rubric
        activate Main API
        Main API --) Web: Rubric data
        deactivate Main API
    end

    alt Rubric not found
        activate Web
        Web --) Teacher: Create rubric message
        deactivate Web
    else Rubric data
        activate Web
        Web --) Teacher: Rubric UI
        deactivate Web
    end
```

## 2. Grade the laboratory

Supposing that the rubric exists, the teacher can grade the laboratory by selecting a criteria for each objective:

```mermaid
sequenceDiagram
    Teacher -)+ Web:        Select student to grade laboratory
    Web -)+ Main API:       Get laboratory rubric
    Main API --)- Web:      Rubric data
    Web --)- Teacher:       Rubric UI
    Web -)+ Main API:       Get student's grade data
    Main API -)+ Database:  Get student's grade data

    alt Grade not found
        Main API -) Database: Create student's grade
    end

    Database --)- Main API:     Student's grade data
    Main API --)- Web:          Student's grade data
    Note LEFT of Web:           Grade UUID is saved for further request

    Teacher -)+ Web:    Selects a criteria for an objective
    Web -)+ Main API:   Add criteria to student's grade
    Note RIGHT of Web:  Grade UUID is sent to identify the grade

    Main API -)+ Database:      Get current selected criteria
    Database --)- Main API:     Current criteria

    alt No current criteria
        Main API -)+ Database:  Save criteria selected by teacher
        Database --)- Main API: Response
    else There is a current criteria
        Main API -)+ Database:  Update to criteria selected by teacher
        Database --)- Main API: Response
    end

    Main API --)- Web:  Criteria was set or updated
    Web --)- Teacher:   Confirmation alert
```
