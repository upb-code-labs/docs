# Entity relationship diagram 🎨

```mermaid
erDiagram
    users }o--|| roles:                 "Have role"
    users ||--o{ class_has_users:       "Belong to multiple classes"
    users ||--o{ submissions:           "Make submissions"
    users ||--o| grades:                "Have grades"

    classes }o--|| users:               "Have teacher"
    classes }o--|| colors:              "Have color"
    classes ||--o{ class_has_users:     "Have students"
    classes ||--o{ laboratories:        "Have laboratories"

    laboratories ||--o{ markdown_blocks:    "Have instructions"
    laboratories ||--o{ test_blocks:        "Have tests"
    laboratories ||--o|	rubrics:            "Can have a rubric"
    grades }o--|| laboratories:             "Belong a laboratory"


    test_blocks  }o--|| languages:      "Have programming language"
    submissions }o--|| test_blocks:    "Belong to a test"

    rubrics ||--|{	objectives:         "Have one or more objectives"
    objectives ||--|{ criteria:         "Have one or mor criteria"

	roles {
        UUID            id          "PK; AUTO"
        VARCHAR(16)     name        "NOT NULL; UNIQUE"
    }

    users {
        UUID            id                  "PK; AUTO"
        UUID            role_id             "FK; REFERENCES roles.id"
        VARCHAR(16)     institutional_id    "NOT NULL; UNIQUE"
        VARCHAR(64)     email               "NOT NULL; UNIQUE"
        VARCHAR(255)    full_name           "NOT NULL"
        VARCHAR(255)    password_hash       "NOT NULL"
    }

    classes {
        UUID            id                  "PK; AUTO"
        UUID            teacher             "FK; REFERENCES users.uuid"
        UUID            color_id            "FK; REFERENCES colors.uuid"
        CHAR(8)         invitation_code     "NOT NULL; UNIQUE"
        VARCHAR(255)    name                "NOT NULL"
    }

    colors {
        UUID        id              "PK; AUTO"
        CHAR(7)     hexadecimal     "NOT NULL; UNIQUE"
    }

    class_has_users {
        UUID        class_id            "FK; REFERENCES classes.id"
        UUID        user_id             "FK; REFERENCES users.id"
        BOOLEAN     is_class_hidden     "DEFAULT FALSE"
        BOOLEAN     is_user_active      "DEFAULT TRUE"
    }

    laboratories {
        UUID                id              "PK; AUTO"
        UUID                class_id        "FK; REFERENCES classes.id"
        UUID                rubric_id       "FK; DEFAULT NULL; REFERENCES rubrics.id"
        VARCHAR(255)        name            "NOT NULL"
        Timestamp           opening_date    "NOT NULL"
        Timestamp           due_date        "NOT NULL"
    }

    markdown_blocks {
        UUID            id              "PK; AUTO"
        UUID            laboratory_id   "FK; REFERENCES laboratories.id"
        VARCHAR()       content         "NULL"
        Uint            index           "NOT NULL; DEFAULT 0"
    }

    test_blocks {
        UUID            id              "PK; AUTO"
        UUID            laboratory_id   "FK; REFERENCES laboratories.id"
        UUID            language        "FK; REFERENCES languages.uuid"
        VARCHAR(255)    name            "NOT NULL"
        BLOB            tests_archive   "NOT NULL"
        Uint            index           "NOT NULL; DEFAULT 0"
    }

    languages {
        UUID            id              "PK; AUTO"
        VARCHAR(32)      name            "NOT NULL; UNIQUE"
        BLOB             base_archive    "NOT NULL"
    }

    submissions {
        UUID            id                  "PK; AUTO"
        UUID            test_id             "FK; REFERENCES test_blocks.id"
        UUID            student_id          "FK; REFERENCES users.id"
        BLOB            archive             "NOT NULL"
        BOOLEAN         passing             "DEFAULT FALSE"
        VARCHAR(16)     status              "DEFAULT 'pending'; ENUM ['pending', 'running', 'ready']"
        VARCHAR()       stdout              "DEFAULT NULL"
    }

    rubrics {
        UUID            id              "PK; AUTO"
        UUID            teacher_id      "FK; REFERENCES users.id"
        VARCHAR(255)    name            "NOT NULL"
    }

    objectives {
        UUID            id              "PK; AUTO"
        UUID            rubric_id       "FK; REFERENCES rubrics.id"
        VARCHAR(255)    name            "NOT NULL"
    }

    criteria {
        UUID                id              "PK; AUTO"
        UUID                objective_id    "FK; REFERENCES objectives.id"
        VARCHAR()           description     "NOT NULL"
        DECIMAL()           value           "NOT NULL"
        Timestamp   created_at      "DEFAULT NOW"
    }

    grades {
        UUID        id                  "PK; AUTO"
        UUID        laboratory_id       "FK; REFERENCES laboratory.id"
        UUID        student_id          "FK; REFERENCES users.id"
        DECIMAL()   score               "NOT NULL"
    }
```

## Design notes 🤔

- `Laboratories` are made up of `markdown blocks` to provide instructions to the students and `test blocks` to test the code written by the students.

- The `markdown blocks` are used by teachers to provide instructions to the students.

- The `test blocks` are used to test the code written by the students.

- The `colors` table is used to easily modify the color scheme of the system. Each class should have a randomly chosen color from the `colors` table associated with it.

- The `languages` table stores the programming languages that can be used to write the code for the test blocks. The only supported language will be Java for now, but **the system needs to be able to support multiple languages in the future**.

- The `base_archive` field in the `languages` table is a zip file containing the base code that will be used by the teachers to write the tests and by the students to write their code. This archive will be defined by the programmers.

- Add an `UNIQUE` constraint to the `submissions` table to prevent students from submitting code to the same tests multiple times. `UNIQUE (test_id, student_id)`.
