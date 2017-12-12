# NCrunch log parser

[NCrunch](http://www.ncrunch.net/) is a concurrent test runner that aims to build and run tests as code changes.

This R code scans log files output either via the Visual Studio GUI or the NCrunch.exe console application, and allows you to visualise whether tests are being run concurrently or not.

You can see a sample here:
* [Sample output in markdown format](./Sample/Readme.markdown).

# Usage

Currently, requires manual interaction. Ideally, more of this would be automated at some stage.

## 1. Install prerequisites
Install R and RStudio if you don't already have them. This step assumes that you have [Chocolatey](https://chocolatey.org/install) installed.

```powershell
choco install -y microsoft-r-open r.studio
```

Install the latest NCrunch for Visual Studio.

## 2. Gather data on resource restrictions

You'll need information on any restrictions you've placed on the NCrunch engine, such as via `[Serial]` attributes. To get this information, you'll need to open the NCrunch test window in Visual Studio with your solution loaded and follow these steps:

1. Make sure all tests are shown in the NCrunch Tests window, including passing tests.
2. Make sure that the `Exclusively Used Resources` and `Full Test Name` columns are visible.
3. Export the NCrunch Tests window in CSV format, saving it to a file called `NCrunchTestsWindow.csv` under a directory called `Data` in this repository.

I'm unable to find any way to get this information from the `NCrunch.exe` console application. Note that you won't need to regenerate this data on every test run since it typically won't change very often.

## 3. Gather test run data

You can run the tests in the Visual Studio GUI or via the NCrunch.exe console application. Follow the appropriate steps here.

### 3.1 Running tests via the Visual Studio GUI

1. Make sure your solution is up-to-date and built.
2. Configure NCrunch to output logs with "Detailed" or "Medium" verbosity to the NCrunch Diagnostic Visual Studio output window.
3. Clear the NCrunch Diagnostic Output menu.
4. Run the resynchronise NCrunch task for a complete test run (make sure that the NCrunch engine mode is set to "Run all tests automatically"), or run the subset of tests that you want to view the performance of.
5. Wait for the test run to complete.
6. Use "File > Save Output As" to save the log output, saving the file in the `Data` directory you created earlier with the name based on the current date and time `<YYYY-MM-DD HH-MM-SS> NCrunchDiagnosticOutput - GUI.txt`.


### 3.2 Running tests via the NCrunch.exe console runner

1. Make sure your solution is up-to-date and built.
2. Run the following PowerShell command from within the root of this cloned repository, updating the `<SolutionPath>` to the absolute path to your Visual Studio solution:
    ```powershell
    & "C:\Program Files (x86)\Remco Software\NCrunch Console Tool\NCrunch.exe" <SolutionPath> -LogVerbosity Detailed -MaxNumberOfProcessingThreads 5 | Out-File -Encoding UTF8 "Data\$(Get-Date -Format "yyyy-MM-dd HH-mm-ss") NCrunchLog - console.txt"
    ```

## 4. Analyse log files

Open `NCrunchLogParser.RProj` in RStudio and open `NCrunch build log parser.Rmd` from the `Files` view. Hit Ctrl-Alt-R to run the script. This will parse the most recent log file in the Data directory and generate the appropriate graphs.

It is also possible to output HTML, PDF, or markdown files from R, but you may require further dependencies. Try with Ctrl-Shift-K.

## Quality

Note that this is a diagnostic tool, so there are likely to be bugs in both these instructions and the script itself. Please contribute back any improvements you may make. There are also several assumptions make in the script, so you'll need to review it to make sure that it makes sense in your case e.g. your local timezone.
