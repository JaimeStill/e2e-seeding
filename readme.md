# Cypress Testing with Disposable Data API

> Still a work in progress.

* [Overview](#overview)
* [Relevant Infrastructure](#relevant-infrastructure)
* [Walkthroughs](#walkthroughs)
    * [Cypress Walkthrough](#cypress-walkthrough)
    * [Angular Rig Test Walkthrough](#angular-rig-test-walkthrough)
    * [Rig Server API Walkthrough](#rig-server-api-walkthrough)
* [Notes](#notes)
    * [SQL Server Express](#sql-server-express)
    * [Cypress Configuration](#cypress-configuration)
    * [Cypress Asynchronous Tasks](#cypress-asynchronous-tasks)

## Overview
[Back to Top](#cypress-testing-with-disposable-data-api)

Given a .NET 6 API using EF Core with SQL Server and a web client, create an API-accessible server that allows a client service to initialize disposable data state from a client service. When executing Cypress tests, the data state can be initialized before all tests are run, then disposed of after all tests are completed.

https://user-images.githubusercontent.com/14102723/190814410-96f3d5d9-3098-44b0-9d20-aa75142303c4.mp4

https://user-images.githubusercontent.com/14102723/188249836-c5e70ac9-ba28-4851-8f1f-47cdebaea7e7.mp4

## Relevant Infrastructure
[Back to Top](#cypress-testing-with-disposable-data-api)

* [Rig Server](./server/Brainstorm.Rig)
    * [DbManager.cs](./server/Brainstorm.Data/DbManager.cs) - Connections are generated from [connections.json](./server/Brainstorm.Data/connections.json)
    * [ProcessRunner.cs](./server/Brainstorm.Rig/Services/ProcessRunner.cs)
    * [ApiRig.cs](./server/Brainstorm.Rig/Services/ApiRig.cs) - Registered as a Singleton service in [Program.cs](./server/Brainstorm.Rig/Program.cs#L34). Because it is created by the dependency injection container, it will properly execute the `IDisopsable.Dispose` method, cleaning up all session resources.
    * [RigController.cs](./server/Brainstorm.Rig/Controllers/RigController.cs)
* [Rig Client](./app/src/rig) - Written in native TypeScript to be framework agnostic
    * [rig.ts](./app/src/rig/rig.ts)
    * [rig-state.ts](./app/src/rig/rig-state.ts)
    * [rig-socket.ts](./app/src/rig/rig-socket.ts)
    * [rig-output.ts](./app/src/rig/rig-output.ts)    
* [test.route.ts](./app/src/brainstorm/app/routes/test/test.route.ts) - Facilitates Rig API interaction testing outside of the context of Cypress.
* [cypress](./app/src/brainstorm/cypress)
    > This infrastructure layout is still being worked on. This will be refactored to facilitate code reuse and minimize code duplication.
    * [test](./app/src/brainstorm/cypress/test/) - Defines route-specific tests and aggregates their execution under a single `Test` class.
        * [home.ts](./app/src/brainstorm/cypress/test/home.ts) - Defines functions for executing tests against 'http://localhost:3000', using the [Rig](./app/src/rig/rig.ts) client to generate / dispose data state.
        * [index.ts](./app/src/brainstorm/cypress/test/index.ts) - Exposes a `Test` class that provides methods pointing to the `test` method of each internal test class.
    * [e2e](./app/src/brainstorm/cypress/e2e) - Cypress test root directory
        * [home.cy.ts](./app/src/brainstorm/cypress/e2e/home.cy.ts) - Executes `Test.home()` as defined in the [test](./app/src/brainstorm/cypress/test/index.ts) directory.

## Walkthroughs
[Back to Top](#cypress-testing-with-disposable-data-api)

Before executing, make sure to install dependencies and build:

```bash
cd /server/
dotnet build

cd ../app/
npm i
npm run build
```

Also be sure to see the [SQL Server Express](#sql-server-express) section.

Connection strings are defined at:

* [appsettings.Development.json](./server/Brainstorm.Api/appsettings.Development.json)
* [connections.json](./server/Brainstorm.Data/connections.json)

URLs:

Asset | URL | Start
------|-----|------
[Brainstorm.Api](./server/Brainstorm.Api/) | http://localhost:5000 | `/server/Brainstorm.Api > dotnet run`
[Brainstorm.Rig](./server/Brainstorm.Rig/) | http://localhost:5001 | `/server/Brainstorm.Rig > dotnet run`
[Angular App](./app/src/brainstorm) | http://localhost:3000 | `/app > npm run start`

### Cypress Walkthrough
[Back to Top](#cypress-testing-with-disposable-data-api)

> Screenshots / videos still need to be generated.

In VS Code, open three different terminals:

**Terminal 1**

```bash
cd /server/Brainstorm.Rig/
dotnet run
```

**Terminal 2**

```bash
cd /app/
npm run start
```

**Terminal 3**

```bash
cd /app/
npm run e2e-open
```

### Angular Rig Test Walkthrough
[Back to Top](#cypress-testing-with-disposable-data-api)

> Screenshots / videos still need to be generated.

In VS Code, open two different terminals:

**Terminal 1**

```bash
cd /server/Brainstorm.Rig/
dotnet run
```

**Terminal 2**

```bash
cd /app/
npm run start
```

From a browser, navigate to http://localhost:3000/test.

### Rig Server API Walkthrough
[Back to Top](#cypress-testing-with-disposable-data-api)

> Screenshots are out of date and need to be updated. Still provides a general idea of what the API provides.

1. Starting up the [Brainstorm.Rig](./server/Brainstorm.Rig) project gives access to the API:  

    ![01-dotnet-run](https://user-images.githubusercontent.com/14102723/185768597-417e5772-4f65-48f7-8d88-29f79b054ce3.png)

2. Calling the `InitializeDatabase` API method generates the test database:

    ![02-initialize-database](https://user-images.githubusercontent.com/14102723/185768718-94114c04-75d1-44dd-94df-58211a09f2ba.png)

3. Calling the `StartProcess` API method starts the [Brainstorm.Api](./server/Brainstorm.Api) project pointed to the test connection string:

    ![03-start-process](https://user-images.githubusercontent.com/14102723/185768726-bd39b80e-d31e-44a4-aff9-72e467c7ad8e.png)

4. The [Brainstorm.Api](./server/Brainstorm.Api) project is now available to interact with:

    ![04-api-swagger](https://user-images.githubusercontent.com/14102723/185768760-a2a06e10-6155-43ea-865b-7c11d889eca8.png)

5. Example of adding a Topic:

    ![05-api-post](https://user-images.githubusercontent.com/14102723/185768775-c7f986f3-86a0-49c1-a6b6-fc309e9eb086.png)

6. Example of calling the Topic `Query` endpoint:

    ![06-api-query](https://user-images.githubusercontent.com/14102723/185768787-ecd435a3-6f6f-43c2-800a-f7f723eb11ac.png)

7. When the Rig server process ends, it properly disposes of the Api server process and deletes the test database:

    ![07-end-session](https://user-images.githubusercontent.com/14102723/185768797-09ff7527-53a8-4bb4-8c74-fca5d47a969c.png)

## Notes
[Back to Top](#cypress-testing-with-disposable-data-api)

### SQL Server Express
[Back to Top](#cypress-testing-with-disposable-data-api)

Testing environment runs using [SQL Server 2019 Express](https://go.microsoft.com/fwlink/p/?linkid=866658) with the server name of `DevSql` and Windows authentication.

In SQL Server Management Studio,right-click the server in object explorer and click **Properties**:

* In the **Security** tab, *cross database ownership chaining* is enabled:

    ![image](https://user-images.githubusercontent.com/14102723/190693425-b43870c4-260f-4959-846f-9fb9834972a9.png)

* In the **Advanced** tab, *Enable Contained Databases* is set to `True`:

    ![image](https://user-images.githubusercontent.com/14102723/190814946-7fa19572-7429-42ff-9539-f2c17f2d4382.png)

Additional Links:
* [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16)
* [SQL Server 2019 Cumulative Update](https://support.microsoft.com/en-us/topic/kb5016394-cumulative-update-17-for-sql-server-2019-3033f654-b09d-41aa-8e49-e9d0c353c5f7)

### Cypress Configuration

[**cypress.config.ts**](./app/src/brainstorm/cypress.config.ts)

* `baseUrl: http://localhost:3000` - Required to prevent re-calling the `before` and `after` methods when `cy.visit()` is executed.

* `defaultCommandTimeout: 8000` - Give a larger buffer for API method execution, particularly when executing `rig.startProcess()`.

[**tsconfig.json**](./app/src/brainstorm/cypress/tsconfig.json)

The following settings are required to be able to use the external [Rig](./app/src/rig/rig.ts) client within tests:

```json
{
    "compilerOptions": {
        "target": "ES5",
        "lib": ["ES5", "DOM"],
        "types": [
            "cypress",
            "node"
        ]
    }
}
```

### Cypress Asynchronous Tasks
[Back to Top](#cypress-testing-with-disposable-data-api)

Cypress does not support using [async / await in tests](https://github.com/cypress-io/cypress/issues/1417) and internally handles asynchrony in non-standard ways. Because of this, when executing asyncronous tasks with non-Cypress infrastructure, these calls have to be wrapped in `cy.then(() => )` and resolved with its own `.then(() => )` call. For instance, when initializing the database before executing tests, the following structure must be used within a test:

```ts
// before all tests execute
before(() => {
    // initialize an asynchronous action with Cypress
    cy.then(() =>
        // execute an external Promise
        rig.initializeDatabase().then(() =>
            // log once the promise completes.
            cy.log('Database initialized')
        )
    );
})
```
