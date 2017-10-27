---
layout: post
Integration Testing VS Code's Tasks API
---

### Background
I recently started contributing to the C# extension for VS Code and the first feature I'm taking on is porting the extension from tasks.json to the new Tasks API. Since the extension is already taking advantage of tasks.json I figured that this would be a fairly simple exercise of removing the json generation and then making the tests pass. Unfortunately, the tests available are fairly focused on the contents of the json file, not on the effect that file has on VS Code. As a new contributor, this made me a bit uncomfortable since I may accidentally break some existing behavior without knowing it existed.

I'm starting, then, by filling in some integration test gaps. Specifically, I want to be able to:
 - Ensure the C# tasks are shown for various C# projects
 - Ensure the C# tasks behave as expected in VS Code for those same projects
 - Ensure the C# tasks are not shown for non-C# projects

As I enable the tests above I'm sure I will learn more about the behavior of VS Code and will be able to refine my approach.

The C# extension's tests already support running tests inside of VS Code, but I did not find any examples that actually interacted with the editor. The first thing I want to do is to get the test editor to show the command palette so I can search for tasks. I hoped to find some guidance on the VS Code docs but they simply showed me how to run unit tests in the VS Code test runner.

### Challenges
After considerable exploration I failed to find any interesting testability abstractions in VS Code so I decided to provide my own through an adapter model. Namely, the strategy looks like this:

 - Create a new module called vscodeAdapter which wraps vscode.
  - I may end up making usage-specific vsCodeAdapters. For example, a vscodeTaskProviderAdapter
  - The adapter starts as a pass-through to the VSCode constructs I need access to
 - In my implementation, use the vscodeAdapter in place of vscode
 - Allow the extension to get wired up by vsCode in the test host. This ensures that I'm as close to testing my production code as possible.
 - Expose relevant abstractions and test hooks from the vsCodeAdapter that are needed to validate behavior in tests

### About the Tasks API

While working on this feature I learned about some interesting behaviors of the Tasks API, specifically vscode.workspace.registerTaskProvider. First off, pay attention to the tasks.json file. In my experiments I saw that VS Code ignores registered task providers if a tasks.json file exists. I only observed `provideTasks` be invoked when I commented out the contents of tasks.json. Perhaps the `type` parameter to `registerTaskProvider` may have some influence, but I did not see any behavioral changes when playing with this value.
