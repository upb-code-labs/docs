# Entity relationship diagram 🎨

```mermaid
erDiagram
    classes }o--|| users:               "Have teacher"
    classes }o--|| colors:              "Have color"
    classes ||--o{ class_has_users:     "Have students"
    classes ||--o{ laboratories:        "Have laboratories"
    
    laboratories ||--|{	tasks:          "Have tasks"
    laboratories ||--o|	rubrics:        "Can have a rubric"

    tasks ||--o{ markdown_blocks:       "Have instructions"
    tasks ||--o{ test_blocks:           "Have code steps"

    test_blocks  }o--|| languages:      "Have programming language"
    test_blocks ||--o{ submissions:     "Have submissions"

    users }o--|| roles:                 "Have role"
    users ||--o| grades:                "Have grades"
    users ||--o{ submissions:           "Make submissions"
    users ||--o{ class_has_users:       "Belong to multiple classes"
    
    rubrics ||--|{	objectives:         "Have one or more objectives"
    objectives ||--|{ criteria:         "Have one or mor criteria"
    laboratories ||--o| grades:         "Belong to laboratories"


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
        VARCHAR(255)        name            "NOT NULL"
        Timestamp           opening_date    "NOT NULL"
        Timestamp           due_date        "NOT NULL"
    }

    tasks {
        UUID        id              "PK; AUTO"
        UUID        laboratory_id   "FK; REFERENCES laboratories.id"
        Timestamp   created_at      "DEFAULT NOW"
    }

    markdown_blocks {
        UUID            id              "PK; AUTO"
        UUID            task_id         "FK; REFERENCES tasks.id"
        VARCHAR()       content         "NULL"
        Uint            index           "NOT NULL; DEFAULT 0"
    }

    test_blocks {
        UUID            id              "PK; AUTO"
        UUID            task_id         "FK; REFERENCES tasks.id"
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
        BLOB            submission_archive  "NOT NULL"
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
        VARCHAR(255)    name            "NOT NULL"
    }

    criteria {
        UUID                id              "PK; AUTO"
        UUID                objective_id    "FK; REFERENCES objectives.id"
        Uint                index           "NOT NULL; DEFAULT 0"
        VARCHAR()           description     "NOT NULL"
        DECIMAL()           value           "NOT NULL"
    }

    grades {
        UUID        id                  "PK; AUTO"
        UUID        laboratory_id       "FK; REFERENCES laboratory.id"
        UUID        student_id          "FK; REFERENCES users.id"
        DECIMAL()   score               "NOT NULL"
    }
```

## Design notes 🤔

- `Laboratories` are made up of `tasks` which are made up of `markdown blocks` and `test blocks`.

- The `markdown blocks` are used by teachers to provide instructions to the students.

- The `test blocks` are used to test the code written by the students.

- The `colors` table is used to easily modify the color scheme of the system. Each class should have a randomly chosen color from the `colors` table associated with it.

- The `languages` table stores the programming languages that can be used to write the code for the test blocks. The only supported language will be Java for now, but **the system needs to be able to support multiple languages in the future**.

- The `base_archive` field in the `languages` table is a zip file containing the base code that will be used by the teachers to write the tests and by the students to write their code. This archive will be defined by the programmers.

## Dummy queries 🏗️

### Create a new class

The `color` field can be chosen from the API using the `colors` table since it wont change often.

The `invitation_code` field can be generated in the API using a random string generator. The 8 character length gives enough combinations to avoid collisions but **it's important to make sure that the generated code is unique**.

```sql
CREATE OR REPLACE FUNCTION create_class(
    teacher_id UUID,
    name VARCHAR(255),
    invitation_code VARCHAR(8),
    color_id UUID
)
    RETURNS UUID
    LANGUAGE PLPGSQL
    AS $$
DECLARE
    class_id UUID;
BEGIN
    INSERT INTO classes (teacher, name, invitation_code, color_id)
    VALUES (teacher_id, name, invitation_code, color_id)
    RETURNING id INTO class_id;

    RETURN class_id;
END $$
```

### Create a new laboratory

Creating a new laboratory also creates a default or first tasks to allow the teachers to start writing instructions and tests.

```sql
CREATE OR REPLACE FUNCTION create_laboratory(
    class_id UUID,
    name VARCHAR(255),
    opening_date TIMESTAMP,
    due_date TIMESTAMP
)
    RETURNS RECORD
    LANGUAGE PLPGSQL
    AS $$
DECLARE
    laboratory_id UUID;
    first_task_id UUID;
    result RECORD;
BEGIN
    INSERT INTO laboratories (class_id, name, opening_date, due_date)
    VALUES (class_id, name, opening_date, due_date)
    RETURNING id INTO laboratory_id;

    INSERT INTO tasks (laboratory_id)
    VALUES (laboratory_id)
    RETURNING id INTO first_task_id;

    result := (laboratory_id, first_task_id);
    RETURN result;
END $$
;
```

### Swap the position of two blocks

The position of two blocks can be swapped by changing their `index` field. The teachers should be able to swap the positions of two adjacent blocks by clicking on up and down arrows.

```sql
CREATE OR REPLACE FUNCTION swap_blocks(
    first_block_id UUID,
    second_block_id UUID
)
    RETURNS VOID
    LANGUAGE PLPGSQL
    AS $$
DECLARE
    first_block_index UINT;
    second_block_index UINT;
BEGIN
    SELECT index INTO first_block_index FROM markdown_blocks WHERE id = first_block_id;
    SELECT index INTO second_block_index FROM markdown_blocks WHERE id = second_block_id;

    UPDATE markdown_blocks SET index = second_block_index WHERE id = first_block_id;
    UPDATE markdown_blocks SET index = first_block_index WHERE id = second_block_id;
END $$
;
```
