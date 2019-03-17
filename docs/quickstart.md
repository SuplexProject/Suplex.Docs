# QuickStart, Detail

This document describes the QuickStart SampleApp in detail, including code, the Suplex admin UI, and how to manipulate values for testing different security scenarios.  The code and executables described within can be found at the locations below:

- <a href="https://github.com/SuplexProject/Suplex.UI.Wpf/releases" target="_blank">Suplex WPF UI</a> for the SampleApp
 for editing Suplex FileStore files
- <a href="https://github.com/SuplexProject/Suplex.Sample" target="_blank">Source code</a> for the SampleApp
- <a href="https://github.com/SuplexProject/Suplex.Sample/releases" target="_blank">Build</a> for the SampleApp

## Overview and Assumptions

The Sample is a simple, single-dialogue, WinForms app, with a few features designed to simulate real-World scenarios common to RBAC security concerns. One upfront assumption: you're probably a better WinForms developer than I, and I know a few parts of the code are a less than optimal.  I built the app to be simple enough for anyone to follow, but I'm happy to take a pull request if there are useful suggestions for improvement.  A high-level overview of the construction is as follows: the main application is a single dialog, split into two parts.

