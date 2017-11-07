---
layout: post
title: Integration Testing VS Code's Tasks API
---

### Background
This post picks up from https://therealpiotrp.github.io/2017-10-26-VSCode-Tasks/
You can also see the relavent PR here: https://github.com/OmniSharp/omnisharp-vscode/pull/1826

### Wiring up the Task API
Enabling API-based task discovery in VS Code occurs across a few extensibility points. 

#### package.json
First, you must define the task type in your extensions package.json:

```
"taskDefinitions": [
   {
      "type": "dotnet",
      "required": ["task"]
   }
]
```

This is the initial task definition for the dotnet task. As we experiment further we may decide to provide several tasks [e.g. build, clean, restore, publish, etc] but for the time being this will suffice. What is important here is the `type` property as it will be used in the programatic construction of tasks.

#### vscode.workspace.registerTaskProvider
With the task defined in package.json you can now invoke `vscode.workspace.registerTaskProvider` method to enable task discovery in VS Code. 
```
registerTaskProvider(type: string, provider: vscode.TaskProvider): vscode.Disposable
```

See that `type` parameter? It needs to match the one you defined in your package.json. This fine piece of trivia took some sleuthing to put together, so I hope it helps.

The `provider` parameter is an interface that supports two properties:
```
export interface TaskProvider {
  /**
   * Provides tasks.
   * @param token A cancellation token.
   * @return an array of tasks
   */
  provideTasks(token?: CancellationToken): ProviderResult<Task[]>;

  /**
   * Resolves a task that has no [`execution`](#Task.execution) set. Tasks are
   * often created from information found in the `task.json`-file. Such tasks miss
   * the information on how to execute them and a task provider must fill in
   * the missing information in the `resolveTask`-method.
   *
   * @param task The task to resolve.
   * @param token A cancellation token.
   * @return The resolved task
   */
  resolveTask(task: Task, token?: CancellationToken): ProviderResult<Task>;
}
```
Note, however, that the second property is a performance optimization that has not yet been implemented and should simply be set to `undefined.

`provideTasks` is where the magic happens. This API returns an array of tasks as discovered by your extension. I spent some time experimenting to determine what each of the required parameters should be set to, so I'll share my findings here:
```
/**
 * ~~Creates a new task.~~
 *
 * @deprecated Use the new constructors that allow specifying a target for the task.
 *
 * @param definition The task definition as defined in the taskDefinitions extension point.
 * @param name The task's name. Is presented in the user interface.
 * @param source The task's source (e.g. 'gulp', 'npm', ...). Is presented in the user interface.
 * @param execution The process or shell execution.
 * @param problemMatchers the names of problem matchers to use, like '$tsc'
 *  or '$eslint'. Problem matchers can be contributed by an extension using
 *  the `problemMatchers` extension point.
 */
constructor(taskDefinition: TaskDefinition, name: string, source: string, 
            execution?: ProcessExecution | ShellExecution, 
	    problemMatchers?: string | string[]);

```

* taskDefinition: this is yet a third place where you need to provide the `type` string from package.json. It's unclear to me why we're providing this value in the provider registration and in the providers return value. Can they differ? I have a feeling there is a face-palm coming as the usage becomes more complicated.
* name: as the doc says, this will show up in the UI when you `Ctrl+Shift+P -> 'Run Task'`. The task is displayed as `${task.source}: ${task.name}` as far as I can tell.
* source: as the doc says. See `name` above for details.
* execution: how to execute the task. more on this below.
* problemMatchers: a string array of names of problem matchers to diagnose the task's output.

I wanted to take a second to look at the `execution` parameter, particularly a value of ShellExecution. 
```
/**
 * Creates a process execution.
 *
 * @param commandLine The command line to execute.
 * @param options Optional options for the started the shell.
 */
constructor(commandLine: string, options?: ShellExecutionOptions);
```
See that 2nd parameter, the Options? It looks like this:
```
 /**
 * Options for a shell execution
 */
export interface ShellExecutionOptions {
	/**
	 * The shell executable.
	 */
	executable?: string;

	/**
	 * The arguments to be passed to the shell executable used to run the task.
	 */
	shellArgs?: string[];

	/**
	 * The current working directory of the executed shell.
	 * If omitted the tools current workspace root is used.
	 */
	cwd?: string;

	/**
	 * The additional environment of the executed shell. If omitted
	 * the parent process' environment is used. If provided it is merged with
	 * the parent process' environment.
	 */
	env?: { [key: string]: string };
}
 ```
 It seems to me that this optional parameter is at odds with the `commandLine` parameter. If my commandline is `dotnet build` but my options is
 ```
 {
    executable: "npm",
    shellArgs: [
      "i"
    ]
 }
 ```
then how should the system behave? Perhaps the executable here actually refers to the shell [e.g. Bash] and not to the command line to execute? I'll follow up with the good folks working on VS Code to find out!

### Next Time
We're working out some kinks in our integration test harness, but next time I'll share the approach we took and the benefits we're enjoying!
