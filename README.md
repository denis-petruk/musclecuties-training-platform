# MuscleCuties Training Platform

MuscleCuties is an iOS strength-training and nutrition journal in TestFlight beta. This repository contains its ASP.NET Core backend, PostgreSQL persistence layer, AWS integration template and bilingual public website.

![The MuscleCuties cat flexing in her training studio](services/static/assets/illustrated-studio.jpg)

## Why I built it

I built MuscleCuties for the women in my family. When a workout moves from Tuesday to Thursday, the harder question is what to do with the rest of the week. I wanted to give them back the time spent making those decisions, while leaving them in control of how they train.

That goal shaped the planning engine. It considers the days available for training, previous sessions and recovery when it builds a plan. The app keeps the planned session separate from the work actually completed, so the next decision can reflect what happened in the gym.

## Workout scheduling and session history

A planned workout and a completed session are separate records. The ASP.NET Core API groups rescheduling, exercise swaps and session logging in the workout session routes. The route definitions show which identifiers each operation needs, how the API determines the current user, and what the client receives. [Workout endpoints](services/MuscleCuties.Server/Workout/Session/WorkoutPlanAndSessionEndpointMappings.cs)

## Food diary operations and retry handling

Copying a meal takes two calendar dates and an optional meal type. Selecting breakfast copies one meal. Leaving the meal type open copies the day. The domain rejects missing or identical dates before the request reaches storage. After a successful copy, the use case signals a change for the destination date only if the operation added entries. [Date rule](src/MuscleCuties.Core/Domain/Nutrition/DiaryCopyDates.cs) · [Copy operation](src/MuscleCuties.UseCases/Features/Nutrition/BatchAndCopy/NutritionBatchAndCopyUseCase.cs) · [Tests](tests/MuscleCuties.Core.Tests/DiaryCopyDateTests.cs)

A phone can lose its connection after PostgreSQL commits but before the response arrives. To handle a retry, the backend stores the command key, a hash of the request and its result in the same transaction as the write. Sending the same command again returns the recorded result. Reusing its key with a different payload raises a conflict. [PostgreSQL command executor](src/MuscleCuties.Infrastructure.PostgreSql/Persistence/PostgresIdempotencyExecutor.cs)

## From API Gateway to PostgreSQL

The AWS CloudFormation template routes an API Gateway HTTP API to a supplied Lambda alias. The alias gives the gateway a stable destination as function versions change. Inside the service, ASP.NET Core wires the use cases to EF Core and Npgsql. Running the API requires database and authentication configuration. [CloudFormation template](infra/cloudformation/http-api.yaml) · [API startup](services/MuscleCuties.Server/ApiApplication.cs) · [PostgreSQL schema](src/MuscleCuties.Infrastructure.PostgreSql/Persistence/Schema)

## Run locally

Build the backend and run the included domain and application tests with the .NET 10 SDK. These tests run without PostgreSQL or AWS credentials.

```sh
dotnet build MuscleCuties.slnx -c Release
dotnet test tests/MuscleCuties.Core.Tests -c Release --no-build
```

To browse the bilingual site, open the landing page directly or serve it locally. The four pages share a stylesheet, preserve the chosen language across links and respect reduced-motion preferences. Contact addresses in this copy are placeholders. [Landing page](services/static/index.html)

```sh
python3 -m http.server 8000 --bind 127.0.0.1 --directory services/static
```

Open <http://localhost:8000>.
