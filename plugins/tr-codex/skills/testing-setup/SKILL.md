---
name: testing-setup
description: Analyze and create a testing strategy for Android apps - install testing
  libraries only when requested, set up scoped test infrastructure, create harnesses for unit tests, UI tests,
  screenshot tests, and end-to-end tests.
license: Complete terms in LICENSE.txt
metadata:
  author: Google LLC
  last-updated: '2026-10-28'
  keywords:
  - android
  - testing
  - ui tests
  - screenshot tests
  - coverage
---

## 프로젝트 범위와 실행 근거

기존 구조, DI, pinned 버전과 요청된 작업을 먼저 확인한다. SDK, emulator, Hilt와 앱 의존성을 자동 설치하거나 전체 앱을 재구성하지 않는다. 원본 예제의 버전과 API가 불명확하거나 프로젝트와 다르면 [find-docs](../find-docs/SKILL.md)에 라이브러리, 정확한 버전과 질문을 전달해 공식 API 근거를 확인한다. 도구가 없거나 지원 API를 확인하지 못하면 미확인으로 보고하며 추측한 API를 구현 완료로 제시하지 않는다. 필요한 테스트 종류와 명령만 선택하고 SDK, 기기, 권한이 없으면 실행 미검증으로 남긴다. 정적 코드 확인과 지침 모델 테스트는 Android 빌드, instrumented 테스트나 실기기 성공의 근거가 아니다.

다른 skill 링크는 해당 작업이 필요할 때만 읽는 선택 참고다. 미선택 upstream skill은 배포되지 않았으며 링크 열람은 설치나 실행 승인이 아니다.

## Step 1: Analyze the current testing setup

To understand the testing setup of an existing project, look for these
dependencies in the libs.versions.toml file, or build files:

1. Dependency Injection framework used. For example, Hilt, Koin, Anvil, or vanilla Dagger.
2. Unit (local) testing framework this project uses. For example, JUnit4 or JUnit5.
3. Mocking framework (if any) used for unit tests, and for instrumented and UI tests. For example, Mockito or Mockk.
4. Robolectric. Identify which of these three uses apply:
   1. Used in unit tests to have fakes for platform entities
   2. To run behavior UI tests without a device or emulator. For example, used to run Espresso or Compose tests.
   3. To do screenshot testing with Roborazzi
5. Is the app 100% Compose, Views or hybrid?
6. Behavior UI tests:
   1. Compose Tests (`androidx.compose.ui:ui-test-*`)
   2. Espresso Tests for Views. Check for wrappers such as Kaspresso. Dependencies: `androidx.test.espresso:espresso-core`, `androidx.test:runner`, `androidx.test:rules`.
7. Screenshot tests. Identify which of these types the project uses:
   1. Instrumented (device-based). For example, using Dropshots.
   2. Based on Robolectric, so they run locally. For example, using Roborazzi.
   3. Based on LayoutLib, so they run locally. For example, Paparazzi, or the Compose Preview Screenshot Testing tool.
8. End to end tests (also known as Release Candidate tests) always run on device and use high-level frameworks such as UIAutomator, Appium, or Robotium.
9. Generate a Markdown report with the analysis

## Step 2: Set up Dependency Injection frameworks for testing

Preserve the existing DI approach, including manual DI. Do not install Hilt, Koin, kotlin-inject or compiler dependencies merely because testing is requested. For an explicitly requested DI migration, assess the existing project first and use only the selected framework’s necessary testing integration.

> [!IMPORTANT]
> **Important:** Always consult the documentation of the applicable framework to learn about testing (for example: [Hilt testing guide](references/android/training/dependency-injection/hilt-testing.md), [Koin Instrumented tests](https://insert-koin.io/docs/reference/koin-android/instrumented-testing)).

For instrumented tests, create and configure (by adding
`testInstrumentationRunner` to the build gradle files) a new test runner and
apply the testing rules required by the framework (for example in Hilt, annotate
your test classes with `@HiltAndroidTest` and apply the `HiltAndroidRule`).
Other frameworks use other mechanisms, consult their documentation.

## Step 3: Install frameworks

Unless otherwise specified, respect the current stack of testing frameworks.

If the user requests test infrastructure installation, select only the necessary framework from these options after checking existing versions and the requested test type. Analysis or a single test request does not authorize installing this whole list:

- JUnit4 for local and instrumented tests
- JaCoCo for test coverage
- For UI tests: if the project has views, Espresso. If it's fully Compose, use the Compose Testing APIs.
- Robolectric to run UI Tests
- Compose Preview Screenshot Testing tool for screenshot tests. Check the
  project's AGP version first, then pick the matching setup and follow it
  strictly:

  - **AGP 9.5.0-alpha03 or higher (recommended):** configure an AGP test suite using `testOptions { screenshotTests.create(...) }`, as described in [Screenshot testing with test suites](references/android/studio/preview/compose-screenshot-testing-with-testsuites.md).
  - **Lower AGP versions:** use the [legacy standalone plugin setup](references/android/studio/preview/compose-screenshot-testing.md) with `com.android.compose.screenshot`, and tell the user it's deprecated. Don't upgrade AGP as part of setting up screenshot tests. Mention that test suites are available on 9.5.0-alpha03 or higher, and leave the upgrade as a separate decision.

  Don't combine both setups in the same module. If the project already uses
  the standalone plugin and its AGP version supports test suites, offer to
  migrate it using the tasks and paths mapping table in
  [Screenshot testing with test suites](references/android/studio/preview/compose-screenshot-testing-with-testsuites.md).
- Dropshots for device screenshot tests

- If a mocking framework is necessary, install Mockk (`io.mockk:mockk`). Do
  not install it unless it is clearly necessary.

For requested instrumented screenshot infrastructure, consider Dropshots only if compatible and within the authorized dependency scope.

For requested end-to-end infrastructure that needs system UI, consider UI Automator only within the authorized dependency scope.

## Step 4: Refactor and create fakes for testing

### **Refactor for unit tests**

In the next sections you'll be asked to create tests. If you have dependencies
on Android framework classes, or entities that are not part of the codebase:

- First, use a fake. If it doesn't exist, create an interface for the class
  and a "Default" implementation with the existing code. Add the Fake version
  to the test sourceset (test or androidTest).

- If not possible to use a fake (example: no access to the class or
  interface), mock the dependencies.

### **Refactor for UI tests**

If you need to fake components to make testing easier and faster and more
reliable, replace slow and problematic dependencies with fakes. Use runtime
fakes using the Dependency Injection framework installed to:

- **Simulate** different scenarios with the user (wrong credentials or a password reset flow), with a server (no connection, server down, or bad JSON from the server) or with a platform component (insufficient permissions, no disk space, no front camera available)
- **Improve** speed and reliability (replacing a database with an in-memory database, replacing a repository with an in-memory fake to avoid hitting the network)

## Step 5: Unit testing

Add or review unit tests for the requested behavior and relevant business logic (ViewModels, Repositories, and database-related classes such as DAOs); do not open an app-wide test expansion task.
Don't create unit tests for Activities, Compose layouts, or dependency injection
configuration files.

## Step 6: UI testing

Espresso or Compose UI tests live in the `test` sourceset because they will be
run with Robolectric. If instrumented (emulator or device) tests are requested,
put them in the `androidTest` sourceset.

## Step 7: Test databases (only when the requested behavior needs database integration coverage)

If the database is using SQLite (through a library such as Room or SQLDelight),
create instrumented tests using an in-memory database to make sure that they
work with the SQLite engine on device.

## Step 8: Screenshot tests (only when requested or needed for the changed UI)

Regardless of the framework used, screenshot tests focus on two types of tests:

- Screen-level screenshot tests, where each screen is tested in nine different sizes, combining compact, medium and expanded widths (400, 610, 900 dp) and heights (400, 500 and 1000 dp).
- Screen-level variations. Add a mobile (400x500) screenshot of:
  - All the alternative themes, if used.
  - Font scale set to 1.5.
- Component-level screenshot tests, where each component is tested in different themes and font scales.

Behavior isn't tested with screenshots, but do test different common scenarios
if their UIs change a lot depending on the state. For example, test loading
screens by injecting a loading state to the UI or simulating it with a fake.

## Step 9: UI Behavior tests

Test the UI logic using behavior tests, which ensures that the UIs react as
expected when different states are passed, and when user actions are performed.

### **Compose UI behavior tests**

- Use the ComposeTestRule with a `ComponentActivity` to access resources such as strings.
- Always match with semantic matchers first. If the matcher is too complicated to write (using more than three matchers to find a single element), use `testTag`.
- Always verify state restoration.

### **Views (XML) UI behavior tests**

Use Espresso to match views and interact with them.

## Step 10: Navigation tests (only for navigation in scope)

Create a test suite to verify navigation logic. Include:

- Back handling
- Deeplinks
- Special patterns like "exit through home" with multiple backstacks.

## Step 11: Simulate different window sizes and settings

For Compose layouts, use `DeviceConfigurationOverride` described in "[UI testing
common patterns](references/android/develop/ui/compose/testing/common-patterns.md)" to simulate different window sizes, font scales

## Step 12: End-to-end tests (only for requested journeys)

Create a low number (about 5% of all tests) of end-to-end tests that cover big
user journeys. Use Compose Test APIs or Espresso for that. If you have to access
platform features such as notifications or system UI, use UI Automator.

If you need to take screenshots of the app running in a device, use
[Dropshots](https://raw.githubusercontent.com/dropbox/dropshots/refs/heads/main/README.md). You need a device for screenshot tests when verifying
interaction with the system UI (examples: edge-to-edge rendering, notifications,
picture-in-picture)

## Step 13: Instrumented Screenshot tests

Only when instrumented screenshot infrastructure is requested and Dropshots is selected, add the compatible plugin and rule for the target feature. Otherwise skip this step.

## Step 14: JaCoCo (only for explicitly requested coverage setup)

Install JaCoCo only when coverage setup is explicitly requested; otherwise keep the existing coverage tooling.

- Only when JaCoCo is selected for the requested coverage setup, add the `jacoco` plugin to the modules in that coverage scope. Otherwise leave module plugins unchanged.

## Final touches

- Ask whether to document the findings of the analysis and the changes applied
  to the testing strategy. If the user agrees:

  - If there is an AGENTS.md file present in the project, update it with any
    changes you've made to the testing strategy.

  - If there is no AGENTS.md file, create a new file (docs/testing.md) with
    a description of the testing strategy, including the commands needed to
    run every type of test and where the screenshot reference files live.
    Also create a new AGENTS.md file in the root and create a link to
    docs/testing.md.
