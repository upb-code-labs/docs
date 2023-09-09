# Entity relationship diagram

```mermaid
erDiagram
    users }o--|| roles: "Has role"
    classes }o--|| users: "Has teacher"
    classes }o--|| colors: "Has color"
    classes ||--o{ class_has_students: ""
    users ||--o{ class_has_students: "Student belongs to multiple classes"
    classes ||--o{ laboratories: "Has laboratories"
    laboratories ||--|{	tasks: "Has tasks"
    tasks ||--o{ markdown_blocks: "Has instructions"
    tasks ||--o{ test_blocks: "Has code steps"
    test_blocks  }o--|| languages: "Has programming language"

	roles {
        UUID    id          "PK; AUTO"
        String  name        "NOT NULL; UNIQUE"
        String  description "NOT NULL; UNIQUE"
    }

    users {
        UUID    id                  "PK; AUTO"
        UUID    roles               "FK; REFERENCES roles.id"
        String  institutional_id    "NOT NULL; UNIQUE"
        String  email               "NOT NULL; UNIQUE"
        String  full_name           "NOT NULL"
        String  password_hash       "NOT NULL"
    }

    classes {
        UUID    id                  "PK; AUTO"
        UUID    teacher             "FK; REFERENCES users.uuid"
        UUID    color_id            "FK; REFERENCES colors.uuid"
        String  invitation_code     "NOT NULL; UNIQUE"
        String  name                "NOT NULL"
        String  color               "NOT NULL; HEXADECIMAL"
    }

    colors {
        UUID    id              "PK; AUTO"
        String  hexadecimal     "NOT NULL; UNIQUE"
    }

    class_has_students {
        UUID    class_id    "FK; REFERENCES classes.id"
        UUID    student_id  "FK; REFERENCES users.id"
    }

    laboratories {
        UUID        id              "PK; AUTO"
        UUID        class_id        "FK; REFERENCES classes.id"
        String      name            "NOT NULL"
        Timestamp   opening_date    "NOT NULL"
        Timestamp   due_date        "NOT NULL"
    }

    tasks {
        UUID        id              "PK; AUTO"
        UUID        laboratory_id   "FK; REFERENCES laboratories.id"
        Timestamp   created_at      "DEFAULT NOW"
    }

    markdown_blocks {
        UUID        id              "PK; AUTO"
        UUID        laboratory_id   "FK; REFERENCES laboratories.id"
        String      content         "NULL"
        Uint        index           "NOT NULL; DEFAULT 0"
    }

    test_blocks {
        UUID        id              "PK; AUTO"
        UUID        laboratory_id   "FK; REFERENCES laboratories.id"
        UUID        language        "FK; REFERENCES languages.uuid"
        String      name            "NOT NULL"
        BLOB        tests_archive   "NOT NULL"
        Uint        index           "NOT NULL; DEFAULT 0"
    }

    languages {
        UUID        id              "PK; AUTO"
        String      name            "NOT NULL; UNIQUE"
        BLOB        base_archive    "NOT NULL"
    }
```

## Design notes

- `Laboratories` are made up of `tasks` which are made up of `markdown blocks` and `test blocks`.

- The `markdown blocks` are used by teachers to provide instructions to the students.

- The `test blocks` are used to test the code written by the students.

- The `colors` table is used to easily modify the color scheme of the system. Each class should have a randomly chosen color from the `colors` table associated with it.

- The `languages` table stores the programming languages that can be used to write the code for the test blocks. The only supported language will be Java for now, but **the system needs to be able to support multiple languages in the future**.

- The `base_archive` field in the `languages` table is a zip file containing the base code that will be used by the teachers to write the tests and by the students to write their code. This archive will be defined by the programmers.
