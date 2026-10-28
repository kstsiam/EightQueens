# EightQueens

[![CI](https://github.com/kstsiam/EightQueens/actions/workflows/ci.yml/badge.svg)](https://github.com/kstsiam/EightQueens/actions/workflows/ci.yml)

An interactive teaching project that solves the classic **8-queens puzzle** with three different approaches and visualizes the results, so you can *see* how algorithm choice changes the size of the search space and the running time.

## Learning objectives

After working through this project you should be able to:

* State the 8-queens problem and the constraints that a valid solution must satisfy.
* Explain how restricting the search space (blind placement → one queen per row → permutations) reduces the work by many orders of magnitude.
* Compute the size of each search space.
* Read and interpret empirical timing results, and recognise the limits of the measurement (timer resolution).
* Describe Heap's algorithm for generating permutations.

## Background: the problem

Place 8 queens on an 8×8 chessboard so that no two queens attack each other. A queen attacks along its row, its column, and both diagonals. A placement is therefore valid only if **no two queens share a row, a column, or a diagonal**.

The code checks these constraints in the `CheckSolution` class (`EightQueens/CheckSolution.cs`): `CheckSolution.HasConflict` validates a whole board using the private `CheckHorizontal`, `CheckVertical` and `CheckDiagonal` helpers.

## The three approaches

Each approach prunes the search space further than the last.

| # | Approach | Idea | Search space | Method in `Form1.cs` |
|---|----------|------|--------------|----------------------|
| 1 | **Brute force: blind** | Drop 8 queens on any of the 64 squares | 64!/56! (≈ 1.8 × 10¹⁴) | `FindSolutionBruteForce` |
| 2 | **Brute force: 1 queen per row** | A row can hold only one queen, so choose a column for each row | 8⁸ = 16,777,216 | `FindSolutionBruteForce1onEachRow` |
| 3 | **Permutations** | Each row *and* column holds exactly one queen, so a solution is a permutation of the columns 0–7 | 8! = 40,320 | `FindSolutionPermutations` |

**Key insight:** every added piece of problem knowledge (a row holds at most one queen; a column holds at most one queen) removes whole classes of invalid boards before we ever test them.

### Discussion questions

1. Why is the blind search space 64!/56! and not 64⁸?
2. Why does the permutation approach still need to check diagonals?
3. By what factor does each step shrink the search space?

## Engineering overview

Beyond the teaching material, the repository documents how the code was modernised safely: characterization tests first, then refactoring, then tooling.

### Refactoring

| Change | Details |
|---|---|
| **Conflict checking extracted from the form** | The row, column and diagonal checks moved out of `Form1.cs` (the WinForms UI) into a static `CheckSolution` class in `EightQueens/CheckSolution.cs`. `Form1` now just calls `CheckSolution.HasConflict(board, queenPos)`. The logic can be tested without any UI. |
| **Tests before and during the refactor** | Characterization tests were added first, so behaviour was pinned down before the code moved. Test helpers live in `BoardHelpers`. |
| **Retargeted to .NET Framework 4.6.2** | Previously 4.6.1. |
| **SDK-style projects** | Both projects were converted so that `dotnet build` and `dotnet test` work from the command line, with NuGet lock files (`packages.lock.json`). |

### Toolchain

| Concern | Tool |
|---|---|
| Language / runtime | C#, Windows Forms, .NET Framework 4.6.2 (`net462`) |
| Build | .NET SDK (`dotnet build`), SDK-style projects, NuGet with lock files |
| Unit tests | [xUnit.net](https://xunit.net/) 2.9 on `Microsoft.NET.Test.Sdk` (ported from MSTest) |
| Coverage | [coverlet](https://github.com/coverlet-coverage/coverlet) collector, [ReportGenerator](https://github.com/danielpalme/ReportGenerator) for HTML reports |
| Static analysis | `Microsoft.CodeAnalysis.NetAnalyzers` (CA rules, `latest-recommended`), IDE style rules enforced in the build, `.editorconfig`, `dotnet format`, NuGet vulnerability and deprecation checks |
| Automation | `tools/analyze.ps1` runs every analysis step; GitHub Actions (`.github/workflows/ci.yml`) builds, analyses, tests and uploads coverage on every push and pull request |

See [CODE_ANALYSIS_GUIDELINES.md](CODE_ANALYSIS_GUIDELINES.md) for the rules, how to change them, and the reasoning behind them.

### Tests

The `EightQueens.Tests` project has **10 xUnit tests, all passing** (last run: `dotnet test` on the .NET 10 SDK, 0 failed, 0 skipped):

| Test class | What it checks |
|---|---|
| `CheckSolutionTests` | A known solution has no conflict; two queens on the same row, column or diagonal are detected; queens a knight's move apart are accepted; **exactly 92 of the 8! = 40,320 permutations** are valid solutions. |
| `KnownSolutionsTests` | The reference list holds the 92 distinct known solutions, each has no conflict, the enumerated solutions equal the reference list, and they form the **12 fundamental solutions** (up to rotation and reflection). |

**Coverage** (coverlet, Cobertura, designer code excluded): `CheckSolution` has **100% line and 100% branch coverage**. The whole assembly is about **25% line / 48% branch**, because the WinForms UI in `Form1` and the search loops are not unit tested. They are exercised by running the app.

## Running the application

This is a Windows Forms app (.NET Framework 4.6.2). Open `EightQueens.sln` in Visual Studio and run it.

The UI offers the following modes:

* **Brute Force: blind**
  * Run once for a specific seed.
  * Run the predefined testbench. *Note: the total measured execution time is ~20–30 minutes.*
* **Brute Force: 1 queen for each row**
  * Run the optimized method for a specific seed.
  * Run the predefined testbench.
* **Permutations**
  * Run the permutations method for a specific seed.
  * Run the predefined testbench.
* **Testing mode**
  * Run all testbenches sequentially.

The seed controls the random choices the search makes. Seeds are predefined for the testbenches so results are **reproducible**.

## Building, running and testing

The project is a Windows Forms application targeting .NET Framework 4.6.2, in SDK-style projects.

### Requirements

* Windows
* The .NET SDK (the `dotnet` command), which restores the NuGet packages on the first build
* The .NET Framework 4.6.2 targeting pack (included in Visual Studio's ".NET desktop development" workload, or installed separately as the .NET Framework 4.6.2 Developer Pack)

### Build

From the repository root, run:

```powershell
dotnet build EightQueens.sln
```

Useful variations:

```powershell
dotnet build EightQueens.sln -c Release
```

```powershell
dotnet build EightQueens.sln --no-incremental
```

To build with MSBuild instead (for example from a "Developer PowerShell for VS"), the Visual Studio installation also needs the ".NET SDK" component, because the projects are SDK-style:

```powershell
msbuild EightQueens.sln /restore /p:Configuration=Debug
```

### Run

The executable is written to `EightQueens\bin\Debug\net462\EightQueens.exe` (or `bin\Release` for a Release build):

```powershell
.\EightQueens\bin\Debug\net462\EightQueens.exe
```

### Test

The unit tests (xUnit.net, 10 tests) live in the `EightQueens.Tests` project and check the Queen conflict detection, including that exactly 92 of the 8! = 40,320 permutations are valid solutions. See [Engineering overview](#engineering-overview) for details. Run them from the repository root:

```powershell
dotnet test EightQueens.sln
```

### Code analysis

```powershell
pwsh tools/analyze.ps1
```

This runs the warnings-as-errors build, the format check, the package vulnerability and deprecation checks, and the tests with coverage. CI runs the same script.

### Code coverage

Run the tests with the coverage collector, then turn the resulting Cobertura XML into an HTML report with [ReportGenerator](https://github.com/danielpalme/ReportGenerator) (installed once with `dotnet tool install -g dotnet-reportgenerator-globaltool`):

```powershell
dotnet test --collect:"XPlat Code Coverage" --settings coverage.runsettings
```

```powershell
reportgenerator -reports:"EightQueens.Tests\TestResults\*\coverage.cobertura.xml" -targetdir:"coveragereport" -reporttypes:Html
```

Open `coveragereport\index.html` to browse the report.

## Sample output

**Testing mode** (seeds 1–15, 15 iterations per method):

![Sample testing-mode execution](https://github.com/TsiamDev/EightQueens/blob/master/EightQueens/Resources/sampleTestsExecution.png?raw=true)

**A single solution** found by the *1 queen for each row* method with seed = 10:

![Sample single solution](https://github.com/TsiamDev/EightQueens/blob/master/EightQueens/Resources/sampleSingleTest.png?raw=true)

## Reading the timing results

* The permutations approach has a start-up cost of a few milliseconds to generate all permutations of [0:7]. After that, execution times are negligible, around 5–10 ms.
* Where the time label on the left of the tab control (always visible) shows **0 ms**, the true time is not zero. It is simply below the timer's resolution (about 5–10 ms).

## Going deeper: Heap's algorithm

The permutations approach generates all permutations of N numbers recursively using **Heap's algorithm** (`heapPermutation` in `Form1.cs`). Reference: [GeeksforGeeks: Heap's algorithm for generating permutations](https://www.geeksforgeeks.org/heaps-algorithm-for-generating-permutations/).

## Exercises

1. Run each single-seed mode and compare the displayed times.
2. Run the testbenches and explain the differences in the bar charts using the search-space sizes above.
3. Change the seed and observe how the blind and one-per-row methods vary, while permutations stays nearly constant.
4. Modify the code to count how many boards each method checks before finding a solution.

## Extension ideas

* Add a **DFS with backtracking** approach, which prunes a partial board as soon as it contains a conflict. This should beat the permutations approach.
* Measure elapsed time with higher precision (nanosecond scale), for example with `System.Diagnostics.Stopwatch`.
