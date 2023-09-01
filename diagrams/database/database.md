# Entity relationship diagram

```mermaid
erDiagram
    users }o--|| roles: "Has role"
    classes }o--|| users: "Has teacher"
    classes ||--o{ class_has_students: ""
    users ||--o{ class_has_students: "Student belongs to multiple classes"
    classes ||--o{ laboratories: "Has laboratories"
    laboratories ||--o{ markdown_blocks: "Has text content"
    laboratories ||--o{ test_blocks: "Has tests"
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
        String  invitation_code     "NOT NULL; UNIQUE"
        String  name                "NOT NULL"
        String  color               "NOT NULL; HEXADECIMAL"
    }

    class_has_students {
        UUID    class_id    "FK; REFERENCES classes.id"
        UUID    student_id  "FK; REFERENCES users.id"
    }

    laboratories {
        UUID    id          "PK; AUTO"
        UUID    class_id    "FK; REFERENCES classes.id"
        String  name        "NOT NULL"
    }

    markdown_blocks {
        UUID        id              "PK; AUTO"
        UUID        laboratory_id   "FK; REFERENCES laboratories.id"
        String      content         "NULL"
    }

    test_blocks {
        UUID        id              "PK; AUTO"
        UUID        laboratory_id   "FK; REFERENCES laboratories.id"
        UUID        language        "FK; REFERENCES languages.uuid"
    }

    languages {
        UUID        id          "PK; AUTO"
        String      name        "NOT NULL; UNIQUE"
    }
```

## Design notes

- `Laboratories` are built from blocks of markdown and test code.

- The `markdown blocks` are used by teachers to provide instructions to the students.

- The `test blocks` are used to test the code written by the students.

- The test file is created and uploaded by the teacher. It's not stored in the database but can be accessed from the file system using the `test_blocks.id` as the file name.

- The `languages` table stores the programming languages that can be used to write the code for the test blocks. The only supported language will be Java for now, but **the system needs to be able to support multiple languages in the future**.
