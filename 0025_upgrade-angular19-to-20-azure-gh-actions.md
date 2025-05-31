# 🚀 How to Upgrade Angular v19 to v20 and Deploy to Azure Using GitHub Actions

Upgrading your Angular project from version 19 to 20 is a good move to get the latest features\! This guide will walk you through the upgrade, explain important changes (like the new `browser` folder in your build output), show you how to adjust your ASP.NET Core backend, and update your GitHub Actions workflow for deploying to Azure. Let's make this easy\! 😊

-----

## ✅ Step 1: Upgrade Angular from v19 to v20

First things first, let's get your Angular project to version 20.

1.  **Use the Official Angular Update Guide:**
    The best place for step-by-step instructions is the official Angular Update Guide.
    👉 [https://angular.dev/update-guide?v=19.0-20.0](https://www.google.com/search?q=https://angular.dev/update-guide%3Fv%3D19.0-20.0) (Make sure to select your correct versions\!)

2.  **Run the Update Command in Your Project:**
    Open your computer's terminal (like Command Prompt, or PowerShell) in your Angular project's main folder. Then, run this command:

    ```bash
    ng update @angular/core@20 @angular/cli@20
    ```

      * This command tells the Angular CLI (`ng`) to update your project's Core Angular parts and the CLI tool itself to version 20.
      * It will check for other packages that need updating and try to make necessary code changes for you. Read any messages it shows and confirm if needed.

3.  **Update Other Packages (Optional but Recommended):**
    After the main update, you can check if other Angular-related packages (like Angular Material, etc.) also need updating:

    ```bash
    ng update
    ```

    This will list any other packages that have new versions compatible with Angular 20.

4.  **Test Your App\!**
    After any update, it's super important to run your app locally and test it thoroughly to catch any problems.

-----

## 🔍 What Has Changed in Angular 20's Build Output?

A key change you'll notice (especially with the default "application" builder, common since Angular 17) is how your project files are organized after you build it for production (using `ng build`).

**Introducing the `browser` Folder:**
When you build your Angular app, the output files (like `index.html`, JavaScript, CSS, and images) are now usually placed inside a **`browser`** subfolder within your main build output directory (often called `dist/your-app-name/`).

  * **Old Way (Simplified View):** `dist/your-app-name/index.html`
  * **New Way (Angular v17+):** `dist/your-app-name/browser/index.html`

If you use Angular Universal for Server-Side Rendering (SSR), you'll also see a `server` folder. But for most client-side apps, the `browser` folder is the important one.

**Why this change?**
It helps organize different types of build outputs better and is a more modern approach.

-----

## 📁 What is the `browser` folder?

Simply put, the `browser` folder holds all the compiled files that run your Angular application in the user's web browser.

Inside this `browser/` folder, you'll find:

  * `index.html` (the main page)
  * JavaScript files (like `main.js`, `polyfills.js`)
  * CSS files (like `styles.css`)
  * Your `assets` folder (with images, fonts, etc.)

Because your files are now in this subfolder, your backend server (which sends these files to the browser) needs to know to look there.

-----

## 🛠️ Backend Changes Needed (ASP.NET Core Example)

If you're using an ASP.NET Core backend to serve your Angular app, you need to tell it about this new `browser` subfolder. Your backend was likely serving files directly from `wwwroot/`, but now it needs to look inside `wwwroot/browser/`.

Here’s how to update your `Program.cs` file (or `Startup.cs` for older .NET versions):

```csharp
using Microsoft.Extensions.FileProviders; // Make sure you have this using statement
using System.IO; // And this one for Path.Combine

// ... (other using statements)

var builder = WebApplication.CreateBuilder(args);

// ... (your services configuration)

var app = builder.Build();

// ... (other middleware like UseHttpsRedirection)

// --- Key Changes for Angular Static Files ---

// 1. Tell ASP.NET Core to serve static files from "wwwroot/browser"
app.UseStaticFiles(new StaticFileOptions
{
    // This points to the physical path on the server
    FileProvider = new PhysicalFileProvider(
        Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", "browser")),
    // This means requests to the root of your site (e.g., yoursite.com/styles.css)
    // will look for 'styles.css' inside the 'wwwroot/browser' folder.
    RequestPath = "" 
});

// ... UseRouting(), UseAuthentication(), UseAuthorization() ...

app.MapControllers(); // If you have API controllers

// 2. Ensure your SPA Fallback is correct
// This makes sure that if someone types a URL like "yoursite.com/some-angular-route",
// it still loads your main Angular app page (index.html).
app.MapFallbackToController("Index", "Fallback"); 
// Alternatively, if you don't use a FallbackController:
// app.MapFallbackToFile("/browser/index.html"); // Ensure this path is correct!

app.Run();

// --- FallbackController Example (if you use it) ---
// Create a new C# file for this controller if you don't have one.
// Example: Controllers/FallbackController.cs
/*
namespace YourApiName.Controllers
{
    public class FallbackController : Controller
    {
        public IActionResult Index()
        {
            // This must point to the index.html inside the browser folder
            return PhysicalFile(
                Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", "browser", "index.html"), 
                "text/HTML");
        }
    }
}
*/
```

**The main idea:** Your ASP.NET Core app now needs to serve your Angular `index.html` and other assets from the `wwwroot/browser/` subfolder.

-----

## 🔁 How to Update Your GitHub Actions YAML File

Your GitHub Actions workflow builds your Angular app and then your .NET app. We need to make sure the Angular files (from its `browser` output folder) are copied into your .NET project's `wwwroot/browser` folder *before* the .NET app is published.

Here’s the reliable two-step approach for your YAML file:

1.  **Build Angular to a temporary (staging) location.**
2.  **Copy the contents of the `browser` subfolder from that temporary location into your .NET project's `wwwroot/browser` folder.**

Here’s an example section of your YAML:
*(Remember to adjust paths like `frontend`, `Main/Applications/Lili.Shop.API` to match your project's structure\!)*

```yaml
# Inside your 'build' job, after checking out code and setting up Node.js:

# Step: Install Angular dependencies and build the Angular app
- name: Install dependencies and build Angular app
  # 'working-directory' ensures these commands run in your Angular app's folder
  working-directory: ./frontend 
  run: |
    npm ci --legacy-peer-deps # Or 'npm install'
    # Build Angular to a temporary location.
    # Angular's builder will create its 'browser' subfolder *inside* ClientBuildOutput.
    # The path here is relative to the 'frontend' working directory.
    npx ng build --configuration production --output-path="../Main/Applications/Lili.Shop.API/ClientBuildOutput" --output-hashing=all

# Step: Copy the Angular 'browser' output to the .NET project's wwwroot/browser
- name: Copy Angular browser output to wwwroot
  run: |
    # This command runs from the root of your GitHub workspace (where your repo was checked out)
    # Ensure these paths match your project structure.
    # This example is for a Windows runner (runs-on: windows-latest)
    xcopy /E /I /Y "Main\Applications\Lili.Shop.API\ClientBuildOutput\browser\*" "Main\Applications\Lili.Shop.API\wwwroot\browser\"
    
    # If you were using a Linux runner (runs-on: ubuntu-latest), you would use 'cp' instead:
    # mkdir -p Main/Applications/Lili.Shop.API/wwwroot/browser
    # cp -rT Main/Applications/Lili.Shop.API/ClientBuildOutput/browser/ Main/Applications/Lili.Shop.API/wwwroot/browser/

# Now, when you run 'dotnet publish' for your .NET project,
# it will include these files from 'wwwroot/browser'.
# - name: Publish .NET project
#   run: dotnet publish ./Main/Applications/Lili.Shop.API/YourApi.csproj -c Release -o "${{ github.workspace }}/myapp"
```

**Why use two steps to copy the files?** (First build to `ClientBuildOutput`, then copy to `wwwroot/browser`)

This two-step way is often safer and helps avoid unexpected problems:

1.  **`ng build` likes a fresh start:**
    Think of `ng build` as a painter. It's easiest for a painter to create a masterpiece on a brand new, clean canvas. That clean canvas is your temporary `ClientBuildOutput` folder.
    If the painter tried to paint directly onto a wall that already has old paint or isn't perfectly smooth (this is like your final `wwwroot/browser` folder, which might have old files or tricky settings), the new painting might not turn out right.
    So, building in a fresh, separate folder lets `ng build` do its job perfectly without anything getting in the way.

2.  **Copying is a simple and clear move:**
    Once the beautiful painting (your Angular app files) is finished on the clean canvas (in `ClientBuildOutput/browser`), the `xcopy` command then does a very simple job: it carefully takes the finished painting and hangs it on the final wall (`wwwroot/browser`). This is a straightforward step and much less likely to cause any damage or errors.

So, this two-step method helps make sure:
* Your Angular app is built correctly in a "clean room."
* The finished files are then moved safely and reliably to the exact spot where your .NET backend needs them.

It's like making sure everything is perfect before you put it in its final place!

### 📘 What do the `xcopy` parameters mean (for Windows runners)?

Let’s quickly understand the `xcopy` command used in the example:
`xcopy /E /I /Y "SourcePath\browser\*" "DestinationPath\browser"`

  * **`xcopy`**: A Windows command for copying files and directory trees.
  * **`/E`**: Copies directories and subdirectories, including empty ones. Makes sure you get everything\!
  * **`/I`**: If the destination doesn't exist and you're copying more than one file, `xcopy` will assume the destination is a directory and create it. This is helpful to create `wwwroot\browser` if it's not there.
  * **`/Y`**: Stops `xcopy` from asking you "Are you sure you want to overwrite this file?". It just overwrites any existing files, which is what you want in an automated script.
  * **`"SourcePath\ClientBuildOutput\browser\*"`**: This is the source. The `\*` means "copy all files and folders" that are *inside* the `ClientBuildOutput\browser` folder.
  * **`"DestinationPath\wwwroot\browser\"`**: This is the destination folder.

Together, this command carefully copies your built Angular app into the right spot for your .NET backend.

-----

## ☁️ Deployment to Azure (How it All Fits)

Let's quickly see the big picture for Azure:

1.  **Angular App is Built:** Your GitHub Action builds Angular. The files end up in `ClientBuildOutput/browser/`.
2.  **Files are Copied:** The `xcopy` (or `cp`) step moves these files into your .NET project's `Main/Applications/Lili.Shop.API/wwwroot/browser/` folder within the GitHub runner's workspace.
3.  **.NET App is Published:** The `dotnet publish` command then packages your .NET application. Because the Angular files are now in its `wwwroot/browser` folder, they get included in the publish output (e.g., into the `myapp` folder).
4.  **Deployed to Azure:** The `azure/webapps-deploy` action takes the `myapp` folder and deploys it to Azure. Your .NET app runs from `D:\home\site\wwwroot` on Azure.
5.  **Correct Path on Azure:** Because your published `myapp` contained `wwwroot/browser`, the final path on Azure for your Angular files becomes `D:\home\site\wwwroot\wwwroot\browser\`.
6.  **ASP.NET Core Serves Files:** Your ASP.NET Core app (running from `D:\home\site\wwwroot`) is configured (in `Program.cs`) to look for static files in its own `wwwroot/browser` subfolder. This matches the actual path, so everything works\!

-----

## ✅ Final Thoughts

Upgrading to Angular 20 means your app benefits from the latest improvements. The main hurdle for deployment, especially with a backend like ASP.NET Core, is handling the new `browser` folder output.

By adjusting your ASP.NET Core static file settings and carefully setting up your GitHub Actions workflow to copy files to the correct location, you can deploy your updated Angular 20 application to Azure without a hitch.

Always remember to test locally after making backend changes and use the logs in GitHub Actions to check file paths if something isn't working as expected.

Happy coding, and enjoy Angular 20\! 🚀



