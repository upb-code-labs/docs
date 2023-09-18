# Main API Software Architecture 🏗️

An implementation of the `Hexagonal Architecture` will be used to build the main API. In this case, each package (Eg. `users`, `laboratories`) will have its own `domain`, `application` and `infrastructure` layers.

## Class diagram 📊

```mermaid
classDiagram
    UserRepositoryImplementation ..|> UserRepositoryInterface: Implements
    UserRepositoryInterface *-- User
    UserUseCases *-- UserRepositoryInterface
    UserHTTPControllers *-- UserUseCases

    class User {
        +String UUID
        +String role
        +String institutionalID
        +String email
        +String fullname
        +String passwordHash
    }

    class UserRepositoryInterface {
        +saveAdmin(user: User): void
        +saveTeacher(user: User): void
        +saveStudent(user: User): void
        +getByUUID(userUUID: String): User
        +getAdmins(): List~User~
        +updatePassword(userUUID: String, newPasswordHash: String): void
    }

    class UserUseCases {
        -UserRepositoryInterface repository

        +registerAdmin(user: User): void
        +registerTeacher(user: User): void
        +registerStudent(user: User): void
        +getUserById(userID: String): void
        +getAdmins(): List~User~
        +updatePassword(userID: String, newPasswordHash: String): void
    }

    class UserHTTPControllers {
        -UserUseCases useCases

        +handleRegisterAdmin(req: Request): Response
        +handleRegisterTeacher(req: Request): Response
        +handleRegisterStudent(req: Request): Response
        +handleGetAdmins(req: Request): Response
        +handleUpdatePassword(req: Request): Response
    }


    class UserRepositoryImplementation {
        +saveAdmin(user: User): void
        +saveTeacher(user: User): void
        +saveStudent(user: User): void
        +getByUUID(userUUID: String): User
        +getAdmins(): List~User~
        +updatePassword(userUUID: String, newPasswordHash: String): void
    }
```
