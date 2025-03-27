## 🖨️ Printess Editor Integration with .NET 9 & Angular 19

This guide explains how to integrate **Printess Editor**, a powerful online design editor for print products, into a modern **.NET 9 backend** and **Angular 19 frontend** application.

---

### 📑 Table of Contents

- [❓ Was ist Printess Editor?](#-what-is-printess-editor)
  - [🔑 Key Features](#-key-features)
- [🚀 Integration Overview](#-integration-overview)
  - [1. Get Tokens](#1-get-tokens)
- [🛠️ Backend Setup (ASP.NET Core / .NET 9)](#️-backend-setup-aspnet-core--net-9)
- [🌍 Local Testing with ngrok](#-local-testing-with-ngrok)
  - [🔧 How to expose your local backend using ngrok](#-how-to-expose-your-local-backend-using-ngrok)
  - [⚠️ Note](#️-note)
- [💻 Frontend Setup (Angular 19)](#-frontend-setup-angular-19)
- [🔁 Callback Flow Diagram](#-callback-flow-diagram)
- [🌀 Polling Flow Diagram](#-polling-flow-diagram)
- [🖼️ Screenshot: Printess Editor in Action (LiliShop)](#️-screenshot-printess-editor-in-action-lilishop)
- [📦 C# Backend Code](#-c-backend-code)
- [🧩 Angular Integration (Frontend Code)](#-angular-integration-frontend-code)
  - [📡 printess-signal-r.service.ts](#-printess-signal-rservicets)
  - [🧾 printess-editor.component.ts](#-printess-editorcomponentts)
  - [🖼️ printess-editor.component.html](#️-printess-editorcomponenthtml)
  - [🎨 printess-editor.component.scss](#-printess-editorcomponentscss)
- [🔗 Useful Resources](#-useful-resources)
- [🏷️ Hashtags](#️-hashtags)

---

## ❓ What is Printess Editor?

**Printess Editor** allows end-users to personalize templates directly in the browser and generate production-ready output such as:

- Flyers
- Business cards
- Posters
- Photo books
- Labels

### 🔑 Key Features
- **Web-based**: Embeds easily via iframe.
- **User-friendly**: No design skills needed; drag-and-drop interface.
- **Template-driven**: Dynamic, designer-built templates.
- **API-connected**: Generate output (PDF/PNG) via the Printess API.
- **Async rendering**: Supports both **callback** and **polling** mechanisms.
- **E-commerce friendly**: Ideal for online shops offering customizable products.

---

## 🚀 Integration Overview

You can integrate Printess using:
- A **Shop Token**: for embedding the editor and loading templates.
- A **Service Token**: for rendering output on the backend.
- **Callback URL**: used by Printess to notify your backend when rendering is complete.

### 1. Get Tokens
Log in to [https://editor.printess.com](https://editor.printess.com) and generate:
- Shop Token
- Service Token
- Embed Code for iframe

---

## 🛠️ Backend Setup (ASP.NET Core / .NET 9)

### Install SignalR:
```bash
dotnet add package Microsoft.AspNetCore.SignalR
```

### Create SignalR Hub:
```csharp
public class PrintessHub : Hub { }
```

### Configure in `Program.cs`:
```csharp
builder.Services.AddSignalR();
app.UseEndpoints(endpoints => {
    endpoints.MapHub<PrintessHub>("/api/printessHub");
});
```

### Enable CORS for local testing with ngrok:
```csharp
builder.Services.AddCors(opt => {
    opt.AddPolicy("CorsPolicy", policy => {
        policy.AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials()
              .SetIsOriginAllowed(origin => origin.Contains("ngrok-free.app"));
    });
});
```

---

## 🌍 Local Testing with ngrok

**ngrok** is a developer tool that creates a secure public tunnel from the internet to your local machine.  
It’s especially useful for testing **webhooks** from external services (like Printess callbacks) while developing locally.

We chose ngrok because:

- It’s quick to set up
- It doesn’t require deployment
- It allows secure HTTPS connections to your localhost

### 🔧 How to expose your local backend using ngrok

1. **Install ngrok** (choose one option):
   - Download manually from: https://ngrok.com/download
   - Or via Chocolatey on Windows:
     ```bash
     choco install ngrok
     ```

2. **Authenticate ngrok** (only once):
   ```bash
   ngrok config add-authtoken YOUR_TOKEN
   ```

3. **Start ngrok tunnel for your backend port**  
   If your backend runs on `http://localhost:8080`:
   ```bash
   ngrok http http://localhost:8080
   ```

4. **Copy the generated ngrok URL**  
   You will see output like:
   ```
   https://1234abcd.ngrok-free.app
   ```

   This URL now tunnels traffic to your local backend. Use it in your callback config:
   ```json
   "CallbackUrl": "https://1234abcd.ngrok-free.app/api/printessEditor/printess-callback"
   ```

### ⚠️ Note

> This URL changes every time you restart ngrok unless you're using a **paid plan** with custom reserved domains.

So remember to **update your callback URL** in `appsettings.json` each time you restart ngrok during development.

---

## 💻 Frontend Setup (Angular 19)

### Install SignalR:
```bash
npm install @microsoft/signalr
```

### Embed Printess Editor iframe:
```html
<iframe
  id="printess"
  src="https://editor.printess.com/printess-editor/embed.html"
  width="100%"
  height="600">
</iframe>
```

### Listen to callback messages:
```ts
window.addEventListener("message", (event) => {
  if (event.data?.cmd === "basket") {
    const saveToken = event.data.token;
    // Store token and send render request
  }
});
```

---

## 🔁 Callback Flow Diagram
A visual overview of how the Printess Editor communicates with the backend and frontend using SignalR during asynchronous document rendering.

![Callback Flow Diagram](https://github.com/user-attachments/assets/77c1a63f-0469-4b14-bb22-1b8ddb49d688)


---

## 🌀 Polling Flow Diagram
A step-by-step overview of how the Printess Editor interacts with the backend to render documents using a polling-based approach.

![Polling-Flow-Diagram](https://github.com/user-attachments/assets/f00c6962-a22e-471a-8e70-8fc9c1192b9a)


---

## 🖼️ Screenshot: Printess Editor in Action (LiliShop)

Below is a live example of the **Printess Editor** embedded inside the **LiliShop Admin Panel**.  
It allows administrators to choose a template, edit designs, and render customized print-ready products (PDF or image) using either the Polling or Callback method.

![Printess-Editor-in-LiliShop](https://github.com/user-attachments/assets/f4708e24-d819-48e2-b602-2cccc984c3ae)

---

## 📦 C# Backend Code
Below is the structure of the complete backend implementation written in **C# with .NET 9**. It includes:
- Controllers
- Services
- DTOs
- Models
- SignalR integration
- Polling and Callback handling

<details>
<summary>Click to expand and view the full C# code</summary>
<br>

```csharp
using Lili.Shop.API.Hubs;
using Lili.Shop.API.Infrastructure;
using Lili.Shop.Core.Settings;
using Lili.Shop.Printess.Dtos;
using Lili.Shop.Printess.Models;
using Lili.Shop.Printess.Services;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.SignalR;
using Microsoft.Extensions.Caching.Memory;
using Microsoft.Extensions.Options;
using System.Net.Http.Headers;
using System.Text.Json;
using System.Text.Json.Serialization;

namespace Lili.Shop.API.Controllers
{
    public class PrintessEditorController : BaseApiController
    {
        private readonly IHubContext<PrintessHub> _hubContext;
        private readonly IPrintessService _printessService;

        public PrintessEditorController(
            IPrintessService printessService,
            IHubContext<PrintessHub> hubContext
            )
        {
            _printessService = printessService;
            _hubContext = hubContext;
        }

        [HttpGet("templates")]
        public async Task<IActionResult> GetTemplates()
        {
            var templates = await _printessService.GetTemplatesAsync();
            return Ok(templates);
        }

        [HttpPost("render-document")]
        public async Task<IActionResult> RenderDocument([FromBody] BuildDocumentRequestDto dto)
        {
            try
            {
                var result = await _printessService.BuildDocumentWithPollingAsync(dto);
                Response.Headers.Add("Content-Disposition", $"attachment; filename={result.FileName}");
                return File(result.FileBytes, result.ContentType, result.FileName);
            }
            catch (Exception ex)
            {
                return StatusCode(500, ex.Message);
            }
        }

        [HttpPost("start-callback-job")]
        public async Task<IActionResult> StartCallbackJob([FromBody] BuildDocumentRequestDto dto)
        {
            try
            {
                string jobId = await _printessService.BuildDocumentWithCallbackAsync(dto);
                return Ok(new { jobId });
            }
            catch (Exception ex)
            {
                return StatusCode(500, ex.Message);
            }
        }

        [HttpPost("printess-callback")]
        public async Task<IActionResult> ReceiveCallback([FromBody] PrintessCallbackDto callback)
        {
            Console.WriteLine($"Received callback for Job ID: {callback.JobId}");
            Console.WriteLine($"IsFinalStatus: {callback.IsFinalStatus}, IsSuccess: {callback.IsSuccess}");
            Console.WriteLine($"Callback data: {JsonSerializer.Serialize(callback)}");

            var (result, fileType) = await _printessService.ProcessCallbackResultAsync(callback);

            if (result is not null)
            {
                await _hubContext.Clients.All.SendAsync("PrintessOutputReady", new
                {
                    jobId = callback.JobId,
                    file = Convert.ToBase64String(result.FileBytes),
                    fileName = result.FileName,
                    contentType = result.ContentType,
                    type = fileType
                });

                Console.WriteLine($"SignalR notified with file: {result.FileName}, type: {fileType}");
            }
            else
            {
                Console.WriteLine("No valid file(s) found in callback.");
            }

            return Ok();
        }
    }
}

namespace Lili.Shop.Core.Settings
{
    public class PrintessOptions
    {
        public string CallbackUrl { get; set; } = string.Empty;
        public string ServiceToken { get; set; } = string.Empty;
        public string TemplateName { get; set; } = string.Empty;
    }
}

namespace Lili.Shop.Printess.Services
{
    /// <summary>
    /// https://www.printess.com/kb.html#api-reference/backend-api.html:round-trip
    /// https://api.printess.com/swagger/index.html
    /// </summary>
    public interface IPrintessService
    {
        Task<List<TemplateDto>> GetTemplatesAsync();
        Task<RenderResult> BuildDocumentWithPollingAsync(BuildDocumentRequestDto dto);
        Task<string> BuildDocumentWithCallbackAsync(BuildDocumentRequestDto dto);
        Task<byte[]> CreateZipFromUrlsAsync(List<string> fileUrls, string extension);
        Task<(RenderResult? Result, string? FileType)> ProcessCallbackResultAsync(PrintessCallbackDto callback);
    }

    public class PrintessService : IPrintessService
    {
        private readonly HttpClient _httpClient;
        private readonly PrintessOptions _options;

        public PrintessService(HttpClient httpClient, IOptions<PrintessOptions> options)
        {
            _options = options.Value;
            _httpClient = httpClient;
            _httpClient.BaseAddress = new Uri("https://api.printess.com/");
            _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", _options.ServiceToken);
        }

        public async Task<List<TemplateDto>> GetTemplatesAsync()
        {
            var response = await _httpClient.PostAsync("templates/list", null);
            response.EnsureSuccessStatusCode();

            var result = await response.Content.ReadFromJsonAsync<TemplateListResponse>();

            var templateDtos = new List<TemplateDto>();
            if (result?.UserTemplateSets is not null)
            {
                foreach (var uts in result.UserTemplateSets)
                {
                    foreach (var template in uts.Templates)
                    {
                        templateDtos.Add(new TemplateDto
                        {
                            Name = template.Name,
                            ThumbnailUrl = template.ThumbnailUrl
                        });
                    }
                }
            }

            return templateDtos;
        }

        public async Task<RenderResult> BuildDocumentWithPollingAsync(BuildDocumentRequestDto dto)
        {
            var isImage = dto.OutputFormat.Equals("image", StringComparison.OrdinalIgnoreCase);

            object produceRequest = BuildProduceRequest(dto, isCallback: false);

            var response = await _httpClient.PostAsJsonAsync("production/produce", produceRequest);

            response.EnsureSuccessStatusCode();

            var result = await response.Content.ReadFromJsonAsync<ProduceResponse>();
            if (result is null || string.IsNullOrEmpty(result.JobId))
            {
                throw new Exception("Failed to retrieve JobId");
            }

            var statusRequest = new StatusRequest { JobId = result.JobId };
            StatusResponse? statusResponse = null;

            var timeout = DateTime.UtcNow.AddMinutes(3);
            while (DateTime.UtcNow < timeout)
            {
                var statusResponseRaw = await _httpClient.PostAsJsonAsync("production/status/get", statusRequest);
                statusResponse = await statusResponseRaw.Content.ReadFromJsonAsync<StatusResponse>();

                if (statusResponse?.IsFinalStatus == true)
                {
                    break;
                }

                await Task.Delay(1000);
            }

            if (statusResponse is null || !statusResponse.IsSuccess)
            {
                throw new Exception("Printess failed to render the output.");
            }

            var resultUrls = isImage
                ? statusResponse.Result?.P?.Select(p => p.U).ToList()
                : statusResponse.Result?.R?.Values.ToList();

            if (resultUrls is null || !resultUrls.Any())
            {
                throw new Exception("No output URLs found.");
            }

            using var httpClient = new HttpClient();

            if (resultUrls.Count == 1)
            {
                var fileBytes = await httpClient.GetByteArrayAsync(resultUrls[0]);
                return new RenderResult
                {
                    FileBytes = fileBytes,
                    ContentType = isImage ? "image/png" : "application/pdf",
                    FileName = isImage ? "output.png" : "output.pdf"
                };
            }
            else
            {
                var zipBytes = await CreateZipFromUrlsAsync(resultUrls, isImage ? "png" : "pdf");
                return new RenderResult
                {
                    FileBytes = zipBytes,
                    ContentType = "application/zip",
                    FileName = "outputs.zip"
                };
            }
        }

        public async Task<string> BuildDocumentWithCallbackAsync(BuildDocumentRequestDto dto)
        {
            var isImage = dto.OutputFormat.Equals("image", StringComparison.OrdinalIgnoreCase);
            var actualTemplate = !string.IsNullOrWhiteSpace(dto.SaveToken) ? dto.SaveToken : dto.TemplateName;

            var produceRequest = BuildProduceRequest(dto, isCallback: true);

            var response = await _httpClient.PostAsJsonAsync("production/produce", produceRequest);

            response.EnsureSuccessStatusCode();

            var result = await response.Content.ReadFromJsonAsync<ProduceResponse>();
            if (result is null || string.IsNullOrEmpty(result.JobId))
            {
                throw new Exception("JobId could not be read.");
            }

            return result.JobId;
        }

        private object BuildProduceRequest(BuildDocumentRequestDto dto, bool isCallback)
        {
            // swagger (https://api.printess.com/swagger/index.html) > POST: /production/produce
            var isImage = dto.OutputFormat.Equals("image", StringComparison.OrdinalIgnoreCase);
            var actualTemplate = !string.IsNullOrWhiteSpace(dto.SaveToken) ? dto.SaveToken : dto.TemplateName;
            var mode = isCallback ? "Callback" : "Polling";
            var outputType = isImage ? "png" : "pdf";

            return new
            {
                templateName = actualTemplate,
                origin = $"LiliShop - {mode} {outputType.ToUpper()} ({dto.TemplateName})",
                callbackUrl = _options.CallbackUrl,
                outputType,
                outputFiles = new[]
               {
                    new
                    {
                        outputSettings = new { dpi = 300 },
                        documentName = "output"
                    }
                }
            };
        }

        public async Task<byte[]> CreateZipFromUrlsAsync(List<string> fileUrls, string extension)
        {
            using var httpClient = new HttpClient();
            using var zipStream = new MemoryStream();
            using (var archive = new System.IO.Compression.ZipArchive(zipStream, System.IO.Compression.ZipArchiveMode.Create, true))
            {
                for (int i = 0; i < fileUrls.Count; i++)
                {
                    var fileBytes = await httpClient.GetByteArrayAsync(fileUrls[i]);
                    var entry = archive.CreateEntry($"output-{i + 1}.{extension}", System.IO.Compression.CompressionLevel.Fastest);

                    using var entryStream = entry.Open();
                    await entryStream.WriteAsync(fileBytes);
                }
            }

            zipStream.Position = 0;
            return zipStream.ToArray();
        }

        public async Task<(RenderResult? Result, string? FileType)> ProcessCallbackResultAsync(PrintessCallbackDto callback)
        {
            string? fileType = null;
            RenderResult? result = null;

            if (callback.Result?.P is not null && callback.Result.P.Count > 1)
            {
                fileType = "zip";
                var zipBytes = await CreateZipFromUrlsAsync(callback.Result.P.Select(p => p.U).ToList(), "png");

                result = new RenderResult
                {
                    FileBytes = zipBytes,
                    FileName = "outputs.zip",
                    ContentType = "application/zip"
                };
            }
            else if (callback.Result?.P is not null && callback.Result.P.Count == 1)
            {
                var imageUrl = callback.Result.P.First().U;
                var imageBytes = await new HttpClient().GetByteArrayAsync(imageUrl);

                fileType = "image";
                result = new RenderResult
                {
                    FileBytes = imageBytes,
                    FileName = "output.png",
                    ContentType = "image/png"
                };
            }
            else if (callback.Result?.R is not null && callback.Result.R.Any())
            {
                var pdfUrl = callback.Result.R.First().Value;
                var pdfBytes = await new HttpClient().GetByteArrayAsync(pdfUrl);

                fileType = "pdf";
                result = new RenderResult
                {
                    FileBytes = pdfBytes,
                    FileName = "output.pdf",
                    ContentType = "application/pdf"
                };
            }

            return (result, fileType);
        }
    }
}

namespace Lili.Shop.Printess.Models
{
    public class RenderResult
    {
        public byte[] FileBytes { get; set; } = Array.Empty<byte>();
        public string ContentType { get; set; } = "";
        public string FileName { get; set; } = "";
    }

    public class ProduceResponse
    {
        [JsonPropertyName("jobId")]
        public string JobId { get; set; } = string.Empty;

        [JsonPropertyName("orderId")]
        public string OrderId { get; set; } = string.Empty;
    }

    public class StatusRequest
    {
        [JsonPropertyName("jobId")]
        public string JobId { get; set; } = string.Empty;
    }

    public class StatusResponse
    {
        [JsonPropertyName("isFinalStatus")]
        public bool IsFinalStatus { get; set; }

        [JsonPropertyName("isSuccess")]
        public bool IsSuccess { get; set; }

        [JsonPropertyName("errorDetails")]
        public string? ErrorDetails { get; set; }

        [JsonPropertyName("result")]
        public StatusResult? Result { get; set; }
    }

    /// <summary>
    /// Represents the result of a Printess production status response.
    /// </summary>
    public class StatusResult
    {
        /// <summary>
        /// A dictionary of rendered PDF documents
        /// Key = document name, Value = direct URL to the PDF file
        /// </summary>
        [JsonPropertyName("r")]
        public Dictionary<string, string>? R { get; set; }

        /// <summary>
        /// A list of distribution image files (PNG, JPG, TIF),
        /// if image output was requested during production
        /// </summary>
        [JsonPropertyName("p")]
        public List<DistributionFile>? P { get; set; }
    }

    /// <summary>
    /// Represents a produced image file returned by Printess (PNG, JPG, TIF).
    /// </summary>
    public class DistributionFile
    {
        /// <summary>
        /// The name of the document
        /// </summary>
        [JsonPropertyName("d")]
        public string D { get; set; } = string.Empty;

        /// <summary>
        /// The URL to download the image file
        /// </summary>
        [JsonPropertyName("u")]
        public string U { get; set; } = string.Empty;

        /// <summary>
        /// The index of this image file in the list of produced files
        /// </summary>
        [JsonPropertyName("i")]
        public int I { get; set; }
    }

    public class TemplateListResponse
    {
        [JsonPropertyName("gid")]
        public string Gid { get; set; } = string.Empty;

        [JsonPropertyName("uid")]
        public string Uid { get; set; } = string.Empty;

        [JsonPropertyName("uts")]
        public List<UserTemplateSet> UserTemplateSets { get; set; } = new();
    }

    public class UserTemplateSet
    {
        [JsonPropertyName("uid")]
        public string Uid { get; set; } = string.Empty;

        [JsonPropertyName("e")]
        public string Email { get; set; } = string.Empty;

        [JsonPropertyName("ts")]
        public List<TemplateEntry> Templates { get; set; } = new();
    }

    public class TemplateEntry
    {
        [JsonPropertyName("id")]
        public int Id { get; set; }

        [JsonPropertyName("n")]
        public string Name { get; set; } = string.Empty;

        [JsonPropertyName("turl")]
        public string ThumbnailUrl { get; set; } = string.Empty;

        [JsonPropertyName("bg")]
        public string Background { get; set; } = string.Empty;

        [JsonPropertyName("ls")]
        public DateTime LastSaved { get; set; }

        [JsonPropertyName("directoryId")]
        public int DirectoryId { get; set; }

        [JsonPropertyName("tt")]
        public string TemplateType { get; set; } = string.Empty;
    }
}

namespace Lili.Shop.Printess.Dtos
{
    public class TemplateDto
    {
        public string Name { get; set; } = string.Empty;
        public string ThumbnailUrl { get; set; } = string.Empty;
    }

    public class PrintessCallbackDto
    {
        [JsonPropertyName("jobId")]
        public string JobId { get; set; } = string.Empty;

        [JsonPropertyName("isFinalStatus")]
        public bool IsFinalStatus { get; set; }

        [JsonPropertyName("isSuccess")]
        public bool IsSuccess { get; set; }

        [JsonPropertyName("errorDetails")]
        public string? ErrorDetails { get; set; }

        [JsonPropertyName("result")]
        public StatusResult? Result { get; set; }
    }

    public class BuildDocumentRequestDto
    {
        public string TemplateName { get; set; } = string.Empty;
        public string? SaveToken { get; set; }
        public string OutputFormat { get; set; } = "pdf"; // "pdf" or "image"
    }
}

```
</details>

---

## 🧩 Angular Integration (Frontend Code)

The following files handle the integration of Printess Editor on the frontend using Angular 19 and SignalR.

### 📡 `printess-signal-r.service.ts`

This service establishes a SignalR connection with the backend to receive real-time updates (PDF or image file results) when a document rendering job is complete.

<details>
<summary>Click to expand</summary>

```ts
import { Injectable, signal } from '@angular/core';
import * as signalR from '@microsoft/signalr';
import { environment } from 'src/environments/environment';

@Injectable({ providedIn: 'root' })
export class PrintessSignalRService {
  private hubConnection: signalR.HubConnection;
  baseUrl = environment.apiUrl;

  isLoading = signal<boolean>(false);

  constructor() {
    this.hubConnection = new signalR.HubConnectionBuilder()
      .withUrl(`${this.baseUrl}printessHub`)
      .withAutomaticReconnect()
      .build();
  }

  startConnection() {
    this.hubConnection.start()
      .then(() => console.log('SignalR Connected!'))
      .catch(err => console.error('SignalR Connection Error:', err));

    this.hubConnection.on('PrintessOutputReady', (data) => {
      console.log('Received from SignalR:', data);
      this.isLoading.set(false);

      const base64      = data.file;
      const fileName    = data.fileName;
      const contentType = data.contentType;
      const type        = data.type;

      if (!base64 || !fileName || !contentType) {
        alert('Missing file data from server.');
        return;
      }

      try {
        const byteCharacters = atob(base64);
        const byteNumbers = new Array(byteCharacters.length);

        for (let i = 0; i < byteCharacters.length; i++) {
          byteNumbers[i] = byteCharacters.charCodeAt(i);
        }

        const byteArray = new Uint8Array(byteNumbers);
        const blob      = new Blob([byteArray], { type: contentType });
        const url       = window.URL.createObjectURL(blob);

        const a = document.createElement('a');
        a.href = url;
        a.download = fileName;
        a.click();
        window.URL.revokeObjectURL(url);

        // Show success message
        const icons: any = { pdf: '📄', image: '🖼️', zip: '🗂️' };
        alert(`${icons[type] ?? '✅'} Your ${type.toUpperCase()} is ready!`);
      } catch (err) {
        console.error('Error decoding file:', err);
        alert('File download failed!');
      }
    });
  }

  stopConnection() {
    if (this.hubConnection) {
      this.hubConnection.stop().then(() => {
        console.log('SignalR Disconnected!');
      });
    }
  }

  isConnected(): boolean {
    return this.hubConnection?.state === signalR.HubConnectionState.Connected;
  }
}

```
</details>


---

### 🧾 `printess-editor.component.ts`

This component handles:

- Output format (`pdf` or `image`)
- **Polling** or **Callback** modes
- Interactions with the iframe (Printess Editor)

<details>
<summary>Click to expand the component TypeScript code</summary>

```ts
import { CommonModule } from '@angular/common';
import { HttpClient, HttpClientModule } from '@angular/common/http';
import { Component, inject, OnDestroy, OnInit, signal } from '@angular/core';
import { PrintessSignalRService } from 'src/app/core/services/printess-signal-r.service';
import { environment } from 'src/environments/environment';

interface TemplateDto {
  name        : string;
  thumbnailUrl: string;
}

@Component({
  selector: 'app-printess-editor',
  templateUrl: './printess-editor.component.html',
  styleUrl: './printess-editor.component.scss',
  imports: [CommonModule, HttpClientModule]
})
export class PrintessEditorComponent implements OnInit, OnDestroy {
  shopToken        = 'eyJhbGciOiJSUzI1NiIsImtpZCI6InByaW50ZXNzLXNhYXMtYWxwaGEiLCJ0eXAiOiJKV1QifQ.eyJzdWIiOiJkYmJlYzIzY2Q3ODA0OWNhOTg4M2I0NWQ2MjdiOTcxMyIsImp0aSI6IkJVWHl0b09RMjlwSURzc0dkRnMxMTZqV0tWSzhib29WIiwicm9sZSI6InNob3AiLCJuYmYiOjE3NDI0MTM3NDcsImV4cCI6MjA1Nzc3Mzc0NywiaWF0IjoxNzQyNDEzNzQ3LCJpc3MiOiJQcmludGVzcyBHbWJIICYgQ28uS0ciLCJhdWQiOiJwcmludGVzcy1zYWFzIn0.YjGEt6U_FtrHHnsE_fvGv-Usf2fKdgVhIGl2VpHLvJT1UkmxOuH0DtxvF7z10M9eSc9MAASfVqwEipwrJJCqG3tYOp_1ALBUFsDavq0QivSvDc_2CGKam8TcTfJ9W46zQO9B5-TA2vLhaAOy4O5kV7i6h1afhWwqbo1pPtk_zzgfU0QegN-0NGYD3PEQsuC3JnR7wEeWMzTxjye3FXnsq-lOiCC8dJcOQSSOYo_x7egb9_W_N-4ea30bKgtuLCFx__FWTQeX9YkpXdiwvqXMy9oEGG_xnk_KY_DFBTiwh8exI22AJ-0Fq_zdNb0R_ui6Ss2uL1pyIqlwWH_bfMZAyJa3Kx3YDjj_k88htt19EsDTUQ56Y3UW38g3NGplKlerab_0A59gHBp12cSXFtTkRdJs85rruLgYuhF2a_0Zh5AiUL5iPjSEreWUbNutMJMcagDwiCy9KcAZLd_Wgx-u-70YryfBW6iruQYqi6Q6J3JCN3H7md1NwATLvYGH7EgvwgQhhjQ6kXn91XhDapLmmzlCOX44JHRahT1p3jfvUHY3JRu82Qa6zYU6-w3DQJqrHWpEdbKVpvSxKFUfpI3WgzNZWzIEozceiAv75tWHdaEK67BnA0vzwCO6otV_GHkG2vIPLS5vngGXiizjsI7u8ybHh7PIQvgvqDwzjXd5wGc';
  baseUrl          = environment.apiUrl;
  iframeId         = 'printess';
  isPollingLoading = signal<boolean>(false);
  saveToken        = signal<string | null>(null);
  useCallback      = signal<boolean>(false);                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                // Default to polling
  templates        = signal<TemplateDto[]>([]);
  templateName     = signal<string>('');
  outputType       = signal<'pdf' | 'image'>('pdf');
  _iframeMessageListenerAttached = false;

  printessSignalR = inject(PrintessSignalRService);
  http            = inject(HttpClient);

  constructor() { }

  ngOnInit(): void {
    this.loadTemplateList();
    if (!this.printessSignalR.isConnected()) {
      this.printessSignalR.startConnection();
    }
  }
  ngOnDestroy(): void {
    this.printessSignalR.stopConnection();
  }

  ngAfterViewInit(): void {
    this.loadPrintess();
  }

  get isCallbackLoading() {
    return this.printessSignalR.isLoading();
  }

  loadTemplateList(): void {
    this.http.get<TemplateDto[]>(`${this.baseUrl}printessEditor/templates`)
      .subscribe({
        next: (data) => {
          this.templates.set(data);
          if (data.length > 0) {
            this.templateName.set(data[0].name);
            this.reloadEditor();
          }
        },
        error: (err) => {
          console.error('Failed to load templates', err);
        }
      });
  }

  onTemplateChange(name: string): void {
    this.templateName.set(name);
    this.useCallback.update(() => false);
    this.saveToken.update(() => null);
    this.loadPrintess();
  }

  reloadEditor(): void {
    const iframe = document.getElementById(this.iframeId) as HTMLIFrameElement;

    iframe.onload = () => {
      iframe.contentWindow?.postMessage({
        cmd: "attach",
        properties: {
          templateName   : this.templateName(),
          templateVersion: "draft",
          token          : this.shopToken
        }
      }, "*");
    };

    iframe.src = "https://editor.printess.com/printess-editor/embed.html";
  }

  isButtonDisabled(): boolean {
    return this.useCallback()
      ? this.isCallbackLoading || !this.saveToken()
      : this.isPollingLoading();
  }

  loadPrintess(): void {
    const iframe = document.getElementById(this.iframeId) as HTMLIFrameElement;

    if (!iframe) {
      return;
    }

    if (!this._iframeMessageListenerAttached) {
      window.addEventListener("message", (event) => {
        switch (event.data.cmd) {
          case "back":
            alert("Back to the catalog. Save-Token: " + event.data.token);
            break;
          case "basket":
            console.log('Received saveToken:', event.data.token);
            this.saveToken.set(event.data.token);
            break;
        }
      });
      this._iframeMessageListenerAttached = true;
    }

    iframe.onload = () => {
      iframe.contentWindow?.postMessage({
        cmd: "attach",
        properties: {
          templateName   : this.templateName(),
          templateVersion: "draft",
          token          : this.shopToken
        }
      }, "*");
    };

    iframe.src = "https://editor.printess.com/printess-editor/embed.html";

    // Optional: forward viewport info to iframe https://www.printess.com/kb.html#api-reference/iframe-ui.html:forwarding-the-visual-viewport
    if (window.visualViewport) {
      window.visualViewport.addEventListener("scroll", () => {
        iframe.contentWindow?.postMessage({
          cmd      : "viewportScroll",
          height   : window.visualViewport?.height,
          offsetTop: window.visualViewport?.offsetTop
        }, "*");
      });
    }
  }

  downloadDocument(): void {
    this.isPollingLoading.set(true);

    const requestPayload = {
      templateName: this.templateName(),
      saveToken   : this.saveToken(),
      outputFormat: this.outputType()
    };

    this.http.post(`${this.baseUrl}printessEditor/render-document`, requestPayload, {
      responseType: 'blob',
      observe: 'response'
    }).subscribe(resp => {
      const contentType = resp.headers.get('content-type') ?? '';
      console.log('Content-Type:', contentType);

      // Detect file type
      const isPdf   = contentType.includes('application/pdf');
      const isImage = contentType.startsWith('image/');
      const isZip   = contentType === 'application/zip';

      if (resp.status === 200 && (isPdf || isImage || isZip)) {
        console.log('File download was triggered successfully by backend.');
      } else {
        alert('No valid file received from backend.');
      }

      this.isPollingLoading.set(false);
    }, err => {
      console.error('Error:', err);
      alert('Download failed: ' + err.message);
      this.isPollingLoading.set(false);
    });
  }

  startCallbackJob(): void {
    if (!this.templateName()) {
      alert('Please choose a template!');
      return;
    }

    if (!this.saveToken()) {
      alert('Your design is not saved. Please click the "Add to Basket" button in the editor!');
      return;
    }

    this.printessSignalR.isLoading.set(true);

    const requestPayload = {
      templateName: this.templateName(),
      saveToken   : this.saveToken(),
      outputFormat: this.outputType()     // can be 'pdf' or 'png'
    };

    this.http.post(`${this.baseUrl}printessEditor/start-callback-job`, requestPayload).subscribe({
      next: () => {
        // SignalR will handle loading indicator reset once the job is completed
      },
      error: (err) => {
        this.printessSignalR.isLoading.set(false);
        alert('Failed to start job: ' + err.message);
      }
    });
  }
}

```
</details>

---

### 🖼️ `printess-editor.component.html`

This HTML template defines the UI layout of the Printess Editor page. It includes:

- A control bar with:
  - Template dropdown
  - Output type selector (PDF or image)
  - Toggle buttons for Polling/Callback mode
  - Generate button
- User hint if `saveToken` is missing in callback mode
- Printess iframe embed

<details>
<summary>Click to expand the component HTML</summary>

```html
<!-- Control Bar -->
<div class="control-bar">
  <div class="template-selector">
    <label for="templateSelect">Choose a template:</label>
    <select id="templateSelect" [value]="templateName()" (change)="onTemplateChange($any($event.target).value)">
      <option *ngFor="let t of templates()" [value]="t.name">{{ t.name }}</option>
    </select>
  </div>

  <div class="output-type-selector">
    <label for="outputType">Output type:</label>
    <select id="outputType" [value]="outputType()" (change)="outputType.set($any($event.target).value)">
      <option value="pdf">PDF</option>
      <option value="image">Image</option>
    </select>
  </div>

  <div class="mode-toggle">
    <button (click)="useCallback.set(false)" [class.active]="!useCallback()">Polling</button>
    <button (click)="useCallback.set(true)" [class.active]="useCallback()">Callback</button>
  </div>

  <div class="generate-button">
    <button (click)="useCallback() ? startCallbackJob() : downloadDocument()" [disabled]="isButtonDisabled()">
      {{ useCallback() && isCallbackLoading ? 'Generating via Callback...' :
         !useCallback() && isPollingLoading() ? 'Generating via Polling...' :
         'Generate Document' }}
    </button>
  </div>
</div>

<!-- Hint -->
<div *ngIf="useCallback() && !saveToken()" class="hint text-center">
  ⚠️ Edit and save your design in the Printess Editor first! (click on 'Add to Basket')
</div>

<!-- Embedded Printess Editor -->
<div class="editor-container">
  <iframe title="Printess Editor" width="100%" height="600px" id="printess"
          src="https://editor.printess.com/printess-editor/embed.html">
  </iframe>
</div>

```
</details>

---

### 🎨 `printess-editor.component.scss`

This SCSS file styles the layout of the Printess Editor component.

<details>
<summary>Click to expand the component SCSS</summary>

```scss
.control-bar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 20px;
  padding: 15px;
  background-color: #f5f5f5;
  border-bottom: 1px solid #ccc;
}

.template-selector select {
  padding: 6px 10px;
  font-size: 15px;
  border-radius: 6px;
  border: 1px solid #ccc;
}

button {
  padding: 10px 20px;
  font-size: 15px;
  font-weight: 500;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  background-color: #0078d4;
  color: white;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

button:hover:not(:disabled) {
  background-color: #005fa3;
  transform: translateY(-1px);
}

button:disabled {
  background-color: #b0b0b0;
  cursor: not-allowed;
}

.mode-toggle button {
  background-color: #e0e0e0;
  color: #333;
}

.mode-toggle button.active {
  background-color: #0078d4;
  color: white;
}

.mode-toggle button:hover:not(.active) {
  background-color: #cccccc;
}

.hint {
  margin-top: 10px;
  font-size: 14px;
  color: #d9534f;
  font-style: italic;
}

.editor-container {
  width: 100%;
  border: 1px solid #ccc;
  margin-top: 0.5rem;
}

.output-type-selector {
  margin-bottom: 1em;
}

.output-type-selector label {
  margin-right: 0.5em;
  font-weight: 500;
}

```
</details>

---

## 🔗 Useful Resources

- 📘 **Printess Backend API Documentation**  
  [https://www.printess.com/kb.html#api-reference/backend-api.html](https://www.printess.com/kb.html#api-reference/backend-api.html)

- 🧪 **Swagger API (Printess REST Interface)**  
  [https://api.printess.com/swagger/index.html](https://api.printess.com/swagger/index.html)

- 🎓 **Printess Foundation Course (Academy)**  
  [https://www.printess.com/kb.html#academy/foundation-course.html](https://www.printess.com/kb.html#academy/foundation-course.html)

- 🚀 **Live Project (LiliShop with Printess Integration)**  
  [https://lilishop-bwdfb5azanh0cfa8.germanywestcentral-01.azurewebsites.net](https://lilishop-bwdfb5azanh0cfa8.germanywestcentral-01.azurewebsites.net)

---

## 🏷️ Hashtags

`#Printess` `#Printess-Editor` `#Csharp` `#NET` `#Angular` `#API` `#integration` `#LiliShop`


---
© 2025 LiliShop — Printess Integration Guide

