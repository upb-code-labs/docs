# Entity relationship diagram 🎨

```mermaid
erDiagram
    users ||--o{ courses_has_users:       "Belong to multiple courses"
    users ||--o{ submissions:           "Make submissions"
    users ||--o| grades:                "Have grades"

    courses }o--|| users:               "Have teacher"
    courses }o--|| colors:              "Have color"
    courses ||--o{ courses_has_users:     "Have students"
    courses ||--o{ laboratories:        "Have laboratories"
    courses ||--o|  invitation_codes:   "Have one invitation code"

    laboratories ||--o{ markdown_blocks:    "Have instructions"
    laboratories ||--o{ test_blocks:        "Have tests"
    laboratories ||--o|	rubrics:            "Can have a rubric"
    grades }o--|| laboratories:             "Belong a laboratory"


    test_blocks  }o--|| languages:      "Have programming language"
    submissions }o--|| test_blocks:    "Belong to a test"

    rubrics ||--|{	objectives:         "Have one or more objectives"
    objectives ||--|{ criteria:         "Have one or mor criteria"

    markdown_blocks ||--|| blocks_index:    "Has index"
    test_blocks ||--|| blocks_index:        "Has index"

    test_blocks ||--|| archives:        "Has a test archive"
    submissions ||--|| archives:        "Has an archive to be tested"
    languages ||--|| archives:          "Has a base archive"


    users {
        UUID            id                  "PK; AUTO"
        UUID            created_by          "FK; REFERENCES users.id"
        ENUM            role                "NOT NULL; ONE OF {'admin', 'teacher', 'student'}"
        VARCHAR(16)     institutional_id    "NOT NULL; UNIQUE"
        VARCHAR(64)     email               "NOT NULL; UNIQUE"
        VARCHAR(255)    full_name           "NOT NULL"
        VARCHAR(255)    password_hash       "NOT NULL"
        Timestamp       created_at          "NOT NULL; DEFAULT NOW()"
    }

    courses {
        UUID            id                  "PK; AUTO"
        UUID            teacher             "FK; REFERENCES users.uuid"
        UUID            color_id            "FK; REFERENCES colors.uuid"
        VARCHAR(255)    name                "NOT NULL"
    }

    invitation_codes {
        UUID            course_id           "PK; FK; REFERENCES courses.id"
        CHAR(9)         code                "NOT NULL; UNIQUE"
        Timestamp       created_at          "NOT NULL; DEFAULT NOW()"
    }

    colors {
        UUID        id              "PK; AUTO"
        CHAR(7)     hexadecimal     "NOT NULL; UNIQUE"
    }

    courses_has_users {
        UUID        course_id            "FK; REFERENCES courses.id"
        UUID        user_id             "FK; REFERENCES users.id"
        BOOLEAN     is_class_hidden     "DEFAULT FALSE"
        BOOLEAN     is_user_active      "DEFAULT TRUE"
    }

    laboratories {
        UUID                id              "PK; AUTO"
        UUID                course_id        "FK; REFERENCES courses.id"
        UUID                rubric_id       "FK; DEFAULT NULL; REFERENCES rubrics.id"
        VARCHAR(255)        name            "NOT NULL"
        Timestamp           opening_date    "NOT NULL"
        Timestamp           due_date        "NOT NULL"
    }

    blocks_index {
        UUID            id                  "PK; AUTO"
        UUID            laboratory_id       "FK; REFERENCES laboratories.id"
        SMALLINT        block_position      "NOT NULL"
    }

    markdown_blocks {
        UUID            id              "PK; AUTO"
        UUID            block_index_id  "FK; REFERENCES blocks_index.id"
        UUID            laboratory_id   "FK; REFERENCES laboratories.id"
        VARCHAR()       content         "NULL"
    }

    test_blocks {
        UUID            id                  "PK; AUTO"
        UUID            block_index_id      "FK; REFERENCES blocks_index.id"
        UUID            laboratory_id       "FK; REFERENCES laboratories.id"
        UUID            language_id         "FK; REFERENCES languages.id"
        BLOB            tests_archive_id    "FK; REFERENCES archives.id"
        VARCHAR(255)    name                "NOT NULL"
        Uint            index               "NOT NULL; DEFAULT 0"
    }

    languages {
        UUID             id                 "PK; AUTO"
        UUID             base_archive_id    "FK; REFERENCES archives.id"
        VARCHAR(32)      name               "NOT NULL; UNIQUE"
    }

    archives {
        UUID            id              "PK; AUTO"
        UUID            file_id         "NOT NULL; UNIQUE"
    }

    submissions {
        UUID            id                  "PK; AUTO"
        UUID            test_id             "FK; REFERENCES test_blocks.id"
        UUID            student_id          "FK; REFERENCES users.id"
        UUID            archive_id          "FK; REFERENCES archives.id"
        BOOLEAN         passing             "DEFAULT FALSE"
        ENUM            status              "DEFAULT 'pending'; ONE OF {'pending', 'running', 'ready'}"
        VARCHAR()       stdout              "DEFAULT NULL"
        Timestamp       created_at          "NOT NULL; DEFAULT NOW()"
    }

    rubrics {
        UUID            id              "PK; AUTO"
        UUID            teacher_id      "FK; REFERENCES users.id"
        VARCHAR(255)    name            "NOT NULL"
    }

    objectives {
        UUID            id              "PK; AUTO"
        UUID            rubric_id       "FK; REFERENCES rubrics.id"
        VARCHAR(510)    description     "NOT NULL"
        Timestamp       created_at      "NOT NULL; DEFAULT NOW()"
    }

    criteria {
        UUID                id              "PK; AUTO"
        UUID                objective_id    "FK; REFERENCES objectives.id"
        VARCHAR(510)        description     "NOT NULL"
        DECIMAL()           weight          "NOT NULL"
        Timestamp   created_at              "NOT NULL; DEFAULT NOW"
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
