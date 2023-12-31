# Dummy queries 🏗️

## Classes 📚

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

### Get the students enrolled in a class

```sql
CREATE OR REPLACE FUNCTION get_students_enrolled_in_class(
    class_id UUID
)
    RETURNS TABLE (
        id UUID,
        full_name VARCHAR(255),
    )
    LANGUAGE PLPGSQL
    AS $$
BEGIN
    RETURN QUERY
    SELECT
        users.id,
        users.institutional_id,
        users.full_name
    FROM users
    JOIN class_has_users ON users.id = class_has_users.user_id
    WHERE
        class_has_users.class_id = class_id AND
        class_has_users.is_user_active = TRUE;
END $$
;
```

## Laboratories 🧪

### Create a new laboratory

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
BEGIN
    INSERT INTO laboratories (class_id, name, opening_date, due_date)
    VALUES (class_id, name, opening_date, due_date)
    RETURNING id INTO laboratory_id;

    RETURN laboratory_id;
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

### Get the number of tests completed by a student

```sql
CREATE OR REPLACE FUNCTION count_tests_completed_by_student(
    student_id UUID,
    laboratory_id_param UUID
)
    RETURNS FLOAT
    LANGUAGE PLPGSQL
    AS $$
DECLARE
    passing_tests UINT;
BEGIN
    -- Get the number of passing tests
    SELECT passing_tests INTO passing_tests FROM students_advances_view
    WHERE
        user_id = student_id AND
        laboratory_id = laboratory_id_param;

    RETURN passing_tests;
END $$
;
```

### Get the number of tests in a laboratory

```sql
CREATE OR REPLACE FUNCTION count_tests_in_laboratory(
    laboratory_id_param UUID
)
    RETURNS UINT
    LANGUAGE PLPGSQL
    AS $$
DECLARE
    tests_count UINT;
BEGIN
    -- Get the number of tests
    SELECT COUNT(*) INTO tests_count FROM test_blocks
    WHERE
        laboratory_id = laboratory_id_param;

    RETURN tests_count;
END $$
;
```
