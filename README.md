# SteamCMD JavaScript Interface
This library allows you to access
[SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD) via JavaScript.

This is compatible with Node > v10 on Windows, Linux, or Mac.

## Basic Usage Example
1. Install the package
   ```sh
   npm install steamcmd-interface
   ```

2. Import the class and create a new instance using the `init` function. This
   is an _asynchronous_ function that downloads all the binaries, creates a new 
   instance of SteamCmd, ensures that it can run, and then returns the instance
   ```js
   const { SteamCmd } = require('steamcmd-interface')
   const steamCmd = await SteamCmd.init()
   ```

3. Now you can use the instance to interact with SteamCMD. You can login with
   your user account or anonymously (default), update apps, or run a series of
   commands.

   The `updateApp` function is an async generator that reports on the progress
   of the update as it gets output from the underling SteamCMD process.
   ```js
   // Downloads CS:GO dedicated server to the default install directory.
   for await(const progress of steamCmd.updateApp(740)) {
     // Logs something like "downloading 1.2%"
     console.log(`${progress.state} ${progress.progressPercent}%`)
   }
   ```

## Construction
A new `SteamCmd` object **cannot** be created using the `new` keyword. It will
throw an error. You must use the `SteamCmd.init` async function. This is because
construction is fundamentally asynchronous.

### Options
An options object can be passed to the `SteamCmd.init` function to configure
the behaviour of the instance. The following options are available:
- `binDir`: The path to which the SteamCMD binaries will be downloaded to.
  Defaults to "[the module's base directory]/temp/install_dir/[platform string]"
- `installDir`: To where SteamCMD will download all applications. Defaults to
  "[the module's base directory]/temp/install_dir"
- `username`: The user name to log in as. Defaults to "anonymous"

### Examples
- Changing the install directory to install apps to the current working
  directory.
  ```js
  SteamCmd.init({
    installDir: process.cwd()
  })
  ```
- Setting a user name for downloading purchased games.
  
  **Note** that this will only work if you successfully logged in once and
  SteamCMD has your credentials cached. See the ["Logging In"](#logging-in) 
  section below for more details.
  ```js
  SteamCmd.init({
    username: 'example'
  })
  ```

## Logging In
SteamCmd offers two login-related functions:
- `isLoggedIn` simply tests if the currently saved username is logged in with
  SteamCMD. If this returns true then SteamCMD has access to your library and
  you can run actions related you your library; such as downloading games.
- `login` uses the given username, password, and Steam Guard code to login.
  This will resolve if the login was successful or throw an error if the login
  failed.

### Examples
1. By default on initialisation SteamCmd logs in anonymously, therefore the
   login test returns true.
   ```js
   const steamCmd = await SteamCmd.init()
   console.log(await steamCmd.isLoggedIn()) // Logs "true"
   ```
2. If we initialise with a username that we have never logged in as then the
   login test returns false.
   ```js
   const steamCmd = await SteamCmd.init({
     username: 'example'
   })
   console.log(await steamCmd.isLoggedIn()) // Logs "false"
   ```
3. Logging in with correct credentials.
   ```js
   const steamCmd = await SteamCmd.init({
     // Specifying the username here is not strictly necessary, because the
     // call to "login" below will update the internally saved username. 
     username: 'example'
   })
   await steamCmd.login('example', 'password123', 'AABB2')
   console.log(await steamCmd.isLoggedIn()) // Logs "true"
   ```
4. If we initialise with a username that we have previously logged in with then
   SteamCMD will use the cached credentials to log us in. Therefore we don't
   need to call the "login" function.
   ```js
   const steamCmd = await SteamCmd.init({
     username: 'example'
   })
   console.log(await steamCmd.isLoggedIn()) // Logs "true"
   ```

## Updating Apps (i.e. Downloading Games)
You can download games into the install directory by using the `updateApp`
function. It's an asynchronous generator function that yields an object that
reports on the current progress of the update. If you are logged in then you
will have access to your Steam library.

You have to give `updateApp` the app ID of the game that you want to download.
You can search for your game to get the app ID on
[Steam DB](https://steamdb.info/search/?a=app).

This function also optionally accepts the platform type and bitness of the
application. This will allow you to download, for example, Windows games on a
Mac. If omitted then the platform and bitness of the current operating system
are used. 

### Example
```js
const steamCmd = await SteamCmd.init()

// Downloads Windows 32bit CS:GO dedicated server to the default install
// directory.
for await(const progress of steamCmd.updateApp(740, 'windows', 32)) {
  // Logs something like
  // {
  //   stateCode: '0x61',
  //   state: 'downloading',
  //   progressPercent: 0.65,
  //   progressAmount: 156521223,
  //   progressTotalAmount: 24015919696
  // }
  console.log(progress)
}

// Once the loop above has completed then the app has been successfully
// downloaded
```

## Running Arbitrary commands
You can run a series of commands using the `run` function. It accepts an array
of strings. Each sting must be a command that can be run by SteamCMD. An
exhaustive list of all available commands is available in
[this repository](https://github.com/dgibbs64/SteamCMD-Commands-List/blob/master/steamcmd_commands.txt).

This function is an asynchronous generator function. It yields each line of
output from SteamCMD. It will throw an error if an error occurred.

### Example
```js
const steamCmd = await SteamCmd.init()

// Uninstall the CS:GO dedicated server
const commands = [
  'app_uninstall -complete 740'
]

for await(const line of steamCmd.run(commands)) {
  console.log(line)
}
```

## Debugging
You can enable debug logging where SteamCmd will log each line of output to the
console. There are two ways you can enable debug logging:
- By passing a parameter to the `init` function
  ```js
  // The second parameter enables or disables debug logging. By default it's
  // disabled
  SteamCmd.init({}, true)
  ```
- By setting a class variable to true. This is useful for enabling or disabling
  debug logging after initialisation
  ```js
  const steamCmd = await SteamCmd.init()

  // ... Later ...

  steamCmd.enableDebugLogging = true
  ```

## Resources
- [SteamCMD home page](https://developer.valvesoftware.com/wiki/SteamCMD)
- [All SteamCMD commands](https://github.com/dgibbs64/SteamCMD-Commands-List)
- [Steam DB app search](https://steamdb.info/search/?a=app)