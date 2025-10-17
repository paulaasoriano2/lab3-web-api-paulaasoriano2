Paula Soriano Sánchez 843710

# Lab 3 Complete a Web API -- Project Report

## Description of Changes
The `ControllerTests.kt` file was completed by implementing all missing `SETUP` and `VERIFY` sections for the HTTP methods `POST`, `GET`, `PUT` and `DELETE` within the `EmployeeController`. The purpose was to validate the correct behavior of each method according to the REST principles of safety and idempotency.

In each test, the Mockk framework was used to simulate the behavior of the `EmployeeRepository`, defining expected responses through `every` and controlling non-returning methods with `justRun`. The verification phase employed `verify` statements to ensure that repository methods were called the expected number of times based on each operation’s semantics.

### POST method

The `POST` operation was tested to confirm that it is not safe and not idempotent, meaning that each identical request creates a new resource.

The every block was configured as follows:

```kotlin
every {
    employeeRepository.save(any<Employee>())
} answers {
    Employee("Mary", "Manager", 1)
} andThenAnswer {
    Employee("Mary", "Manager", 2)
}
```

This setup simulated the creation of two different employees when `save()` was called twice, producing IDs 1 and 2. It ensured that consecutive `POST` requests did not overwrite existing data but generated new entries instead.

The verify section contained:

```kotlin
verify(exactly = 2) {
    employeeRepository.save(any<Employee>())
}
```

This verified that the repository’s `save()` method was invoked twice—once per `POST` call—confirming the non-idempotent nature of the operation. No other repository methods were called, as `POST` is not a safe operation.

### GET method

The `GET` operation was tested to confirm that it is both safe and idempotent, meaning that multiple identical requests retrieve data without causing any change in server state.

The `every` configuration simulated two retrieval scenarios:

```kotlin
every {
    employeeRepository.findById(1)
} answers {
    Optional.of(Employee("Mary", "Manager", 1))
}

every {
    employeeRepository.findById(2)
} answers {
    Optional.empty()
}
```

The first setup returned an existing employee for ID 1, while the second returned an empty result for ID 2 to represent a “not found” condition. This configuration allowed verification of both successful and unsuccessful `GET` requests.

The verify block validated the behavior:
```kotlin
verify(exactly = 2) {
    employeeRepository.findById(1)
}

verify(exactly = 0) {
    employeeRepository.save(any<Employee>())
    employeeRepository.deleteById(any())
    employeeRepository.findAll()
}
```

This confirmed that `findById()` was invoked twice and that no other methods capable of modifying data (`save` or `deleteById`) were executed, validating the safe and idempotent nature of `GET`.

### PUT method

The `PUT` operation was tested to demonstrate that it is not safe but idempotent, as it can modify the server state but repeated identical calls result in the same final state.

The setup used the following configuration:
```kotlin
every {
    employeeRepository.findById(1)
} answers {
    Optional.empty()
} andThenAnswer {
    Optional.of(Employee("Tom", "Manager", 1))
}
```

This setup simulated a scenario where the employee did not exist during the first call (creation case) but was present during the second (update case). It reflected the typical `PUT` behavior, where the first call creates a resource and subsequent identical calls update the same entity.

The save operation was mocked as:
```kotlin
every {
    employeeRepository.save(any<Employee>())
} answers {
    Employee("Tom", "Manager", 1)
}
```

This ensured that `save()` returned the same employee instance with the same ID for both requests.

The verify block was defined as:

```kotlin
verify(exactly = 2) {
    employeeRepository.save(any<Employee>())
}
```

This confirmed that `save()` was invoked twice—once for creation and once for update—consistent with the idempotent behavior of `PUT`.

### DELETE method

The `DELETE` operation was tested to verify that it is not safe but idempotent, since it modifies server state but repeated identical calls leave the system unchanged once the resource is deleted.

The setup simulated an employee existing initially and then being removed:

```kotlin
every {
    employeeRepository.findById(1)
} answers {
    Optional.of(Employee("Tom", "Manager", 1))
} andThenAnswer {
    Optional.empty()
}
```

This sequence reflected a resource that existed before the first deletion and was absent after it.

The `deleteById()` method was configured with justRun because it does not return any value:

```kotlin
justRun {
    employeeRepository.deleteById(1)
}
```

The verification ensured correct invocation of repository methods:

```kotlin
verify(exactly = 2) {
    employeeRepository.deleteById(1)
}
verify(exactly = 0) {
    employeeRepository.save(any<Employee>())
    employeeRepository.findById(any())
    employeeRepository.findAll()
}
```

This verified that the delete operation was executed twice and that no save or retrieval methods were called, confirming the idempotent but unsafe nature of `DELETE`.

All tests passed successfully, demonstrating correct functional behavior for each HTTP method according to RESTful conventions.

## Technical Decisions
Mockk was selected to handle repository mocking due to its expressive syntax and compatibility with Kotlin coroutines and Spring Boot testing tools. The combination of `every`, `andThenAnswer`, and `justRun` provided complete control over simulated repository behavior, ensuring predictable test outcomes.

Each verification was carefully aligned with HTTP semantics, allowing the validation of safety and idempotency at the repository interaction level. Code formatting and structural clarity were maintained using Ktlint to ensure compliance with Kotlin style conventions.

## Learning Outcomes
The completion of these tests reinforced understanding of RESTful API properties, particularly the distinction between safe and idempotent operations. It also demonstrated the practical application of mocking frameworks for simulating persistence layers and testing controller logic in isolation.

## AI Disclosure
### AI Tools Used
The only AI tool which was used was ChatGPT.

### AI-Assisted Work
AI assistance was used solely for refining the language, structure, and clarity of documentation. No code, logic, or implementation decisions were generated or altered through AI. The estimated proportion of AI-assisted work represents approximately fifteen percent of the overall report effort, restricted entirely to linguistic enhancement.

### Original Work
The setup and verification steps of each test were manually designed, executed, and validated to ensure functional correctness.