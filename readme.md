# WealthApp — Backend API (ASP.NET Core)

This repository contains the WealthApp backend: an ASP.NET Core 8 Web API that provides portfolio, transaction, asset pricing, and AI-driven what‑if / advice functionality intended to support a Registered Investment Advisor (RIA) workflow and a companion frontend.

---

## Table of Contents

- Project Overview
- Tech Stack
- Architecture Summary
- API Base URL and Endpoint Structure
- DTO Reference (frontend-facing)
- RIA (Frontend) Workflow
- How to Run the Project
- Configuring the LLM (Gemini / Google Generative Language)
- Seeder Guide
- Troubleshooting

---

## Project Overview

What WealthApp is
- A lightweight portfolio management backend that stores assets, portfolios, transactions and price history and exposes REST endpoints consumed by the frontend.
- Includes deterministic AI services and an optional Large Language Model (LLM) integration (Gemini via Google Generative Language API) to generate narrative explanations and advanced what‑if analysis.

RIA workflow context
- Designed to support a Registered Investment Advisor (RIA) or advisor user workflow:
  - Inspect users and their portfolios
  - See portfolio summaries and allocations
  - Create and list transactions
  - Run scenario (what‑if) simulations on asset price moves
  - Request AI-generated advice and transaction explanations to augment advisor decision‑making (non‑advisory tone)

High‑level goals
- Provide a simple, testable backend API to power a frontend dashboard for portfolio visibility and lightweight AI insights.
- Keep LLM usage optional and pluggable through configuration.
- Use a repository + service architecture with EF Core for easy extensibility and unit testing.

---

## Tech Stack

- ASP.NET Core 8 Web API
- Entity Framework Core (Pomelo MySQL provider; `UseMySql` with ServerVersion `8.0.45-mysql`)
- Gemini LLM integration via a client implementation `GeminiLLMClient` (calls Google Generative Language API)
- Repository + Service architecture:
  - Generic repository: `WealthApp.Repositories.Repository<T>`
  - Services: portfolio, transaction, asset price, AI advice, what‑if simulation, LLM client abstractions

---

## Architecture Summary

Top-level folders and responsibilities
- `Controllers/` — HTTP controllers and routes (API surface).
- `Models/` — EF Core entity models mapping to database tables (`User`, `Portfolio`, `Transaction`, `Asset`, `Assetprice`).
- `Dtos/` — Data Transfer Objects used by controllers and the frontend (DTOs for portfolio, transaction responses, what‑if, AI advice, etc.).
- `Services/` — Business logic and domain services:
  - `PortfolioService`, `TransactionService`, `AssetPriceService`, `WhatIfSimulationService`, `AIAdviceService`, `AITransactionExplanationService`.
  - `ILLMClient` interface and `GeminiLLMClient` implementation for LLM access.
- `Repositories/` — Generic repository abstraction and implementation used for basic CRUD.
- `Program.cs` — Dependency injection, DB context registration, Swagger and CORS setup.
- `seed.ps1` — PowerShell script to reseed the development API with sample data.

Key components
- Models — EF Core entities (`Models/*.cs`) map to underlying MySQL tables (`users`, `assets`, `portfolios`, `transactions`, `assetprices`).
- DTOs — `Dtos/` contains request and response DTOs the frontend consumes (examples below).
- Services — encapsulate domain logic (portfolio math, allocation calculations, volatility/trend approximations).
- LLM integration — `ILLMClient` abstracts an LLM client; `GeminiLLMClient` implements the Generative Language API calls and parsing.
- What‑If Simulation Engine — `WhatIfSimulationService` performs scenario simulations, computes value/allocation deltas, and optionally calls the LLM to produce a human‑readable explanation.
- AI Advice / Transaction Explanation — `AIAdviceService` and `AITransactionExplanationService` provide structured/advisory outputs consumed by the frontend.
- Seeder — `seed.ps1` can populate sample users, assets, portfolios, price history and transactions.

---

## API Base URL and Endpoint Structure

Base URL (development)
- https://localhost:7257/api

Endpoints (by controller)

- Users
  - GET `api/Users`
    - Purpose: List users with portfolios and transactions (DTO projection).
    - Response: `UserDto[]` (includes `Portfolios` with `TransactionDto`).
  - GET `api/Users/{id}`
    - Purpose: Get a single user with portfolio and transaction DTOs.
    - Response: `UserDto`
  - POST `api/Users`
    - Purpose: Create a new user (accepts `Models.User` entity in body).
    - Response: Created `User` entity.
  - PUT `api/Users/{id}`
    - Purpose: Update user (accepts `Models.User`).
  - DELETE `api/Users/{id}`
    - Purpose: Delete user.

- Portfolios
  - GET `api/Portfolios/users/{userId}`
    - Purpose: Get all portfolios for a user (DTO projection).
    - Response: `PortfolioDto[]`
  - GET `api/Portfolios/{id}`
    - Purpose: Get a single portfolio with transactions.
    - Response: `PortfolioDto`
  - POST `api/Portfolios`
    - Purpose: Create a portfolio (body: `CreatePortfolioDto`).
    - Notes: Enforces unique `UserId+AssetId`.
    - Response: Created `Portfolio` entity.
  - PUT `api/Portfolios/{id}`
    - Purpose: Update a portfolio (body: `Models.Portfolio`).
  - DELETE `api/Portfolios/{id}`
    - Purpose: Delete a portfolio.
  - GET `api/Portfolios/{id}/summary`
    - Purpose: Return portfolio summary (investment, current value, P/L).
    - Response: `PortfolioSummaryDto`
  - GET `api/Portfolios/user/{userId}/allocation`
    - Purpose: Return allocation percentages by asset type for a user.
    - Response: `PortfolioAllocationDto[]`

- Transactions
  - GET `api/Transactions/portfolio/{portfolioId}`
    - Purpose: List transactions for a portfolio.
    - Response: `TransactionResponseDto[]`
  - GET `api/Transactions/{id}`
    - Purpose: Get a single transaction.
    - Response: `TransactionResponseDto`
  - POST `api/Transactions`
    - Purpose: Create a transaction (body: `CreateTransactionDto` record declared in controller).
    - Request DTO: `{ PortfolioId, Type, Quantity, Price, Fee? }`
    - Response: `TransactionResponseDto` (Created)
    - Notes: Validation enforces positive price/quantity and Type ∈ { "BUY", "SELL" }. Transaction creation uses `TransactionService` to update portfolio totals.
  - PUT `api/Transactions/{id}`
    - Purpose: Update a transaction (`Models.Transaction` in body).
  - DELETE `api/Transactions/{id}`
    - Purpose: Delete a transaction.

- Assets
  - GET `api/Assets`
    - Purpose: List all assets.
    - Response: `Models.Asset[]`
  - GET `api/Assets/{id}`
    - Purpose: Get asset by id.
  - POST `api/Assets`
    - Purpose: Create asset (body: `Models.Asset`).
  - PUT `api/Assets/{id}`
    - Purpose: Update asset.
  - DELETE `api/Assets/{id}`
    - Purpose: Delete asset.

- AssetPrices
  - POST `api/AssetPrices`
    - Purpose: Create a price point for an asset (body: `CreateAssetPriceDto`).
    - Response: Created `CreateAssetPriceDto`/model mapping.
  - GET `api/AssetPrices/latest/{assetId}`
    - Purpose: Get latest price for an asset.
    - Response: `AssetPriceResponseDto` (or underlying entity projection)

- AI Advice
  - GET `api/AI/advice/transactions/{transactionId}/explain`
    - Purpose: Produce an explanation for a transaction using portfolio summary, allocation, trend, volatility, and the `AITransactionExplanationService`.
    - Response: JSON { transactionId, explanation }
  - GET `api/AI/advice/portfolio/{portfolioId}`
    - Purpose: Generate advice for a given portfolio using `IAIAdviceService`.
    - Response: `AIAdviceResponseDto`

- What‑If Simulation
  - GET `api/AI/whatif/debug/llmsettings`
    - Purpose: Return bound `LLMSettings` (for debugging).
  - GET `api/AI/whatif/debug/llmtest`
    - Purpose: Lightweight test call to `ILLMClient` (to check LLM config).
  - POST `api/AI/whatif`
    - Purpose: Run a what‑if simulation.
    - Request: `WhatIfRequestDto` `{ PortfolioId, AssetId? , PercentageChange }`
    - Response: `WhatIfResultDto` (includes `HypotheticalSummary`, `HypotheticalAllocation`, per-asset deltas, allocation deltas, and an explanation text which is either LLM-generated or deterministic fallback)

Notes
- All API routes are under the `api/` prefix as registered in controller attributes.
- In development, Swagger is enabled; use the Swagger UI to explore the endpoints (`/swagger`).

---

## DTO Documentation

Important DTOs consumed by the frontend:

- `UserDto`
  - Properties: `Id`, `Name`, `Email`, `CreatedAt`, `IsDeleted`, `List<PortfolioDto> Portfolios`
  - Used when listing or viewing users with their portfolios and transactions.

- `PortfolioDto`
  - Properties: `Id`, `UserId`, `AssetId`, `TotalQuantity`, `AverageBuyPrice`, `List<TransactionDto> Transactions`
  - Used by frontend to show holdings and associated transactions.

- `PortfolioSummaryDto`
  - Record: `(int PortfolioId, decimal TotalInvestment, decimal CurrentValue, decimal ProfitLoss, decimal ProfitLossPercentage)`
  - Returned by `GET api/Portfolios/{id}/summary` — central for showing P/L and current value.

- `PortfolioAllocationDto`
  - Record: `(string AssetType, decimal Percentage)`
  - Represents allocation percent by asset type (e.g., `Stock`, `Crypto`).

- `TransactionDto` (DTO projection used internally for nested lists)
  - Properties: `Id`, `PortfolioId`, `Type`, `Quantity`, `Price`, `Fee`, `TransactionDate`
  - Used inside `PortfolioDto`.

- `TransactionResponseDto`
  - Record: `(int Id, string Type, decimal Quantity, decimal Price, decimal? Fee, DateTime? TransactionDate)`
  - Returned by transactions endpoints to the frontend.

- `AIAdviceResponseDto`
  - Record: `(int PortfolioId, string OverallSentiment, string RiskLevel, List<PerAssetInsightDto> PerAssetInsights, string SummaryAdvice)`
  - `PerAssetInsightDto` contains `(string AssetLabel, decimal? SevenDayChange, decimal? ThirtyDayChange, string TrendDescription, string VolatilityCategory)`
  - Used by frontend to display AI-generated portfolio insights.

- `WhatIfRequestDto`
  - Record: `(int PortfolioId, int? AssetId, decimal PercentageChange)`
  - Request body for what‑if simulation; `AssetId` is optional — if omitted the percentage change is applied to all holdings.

- `WhatIfResultDto`
  - Record containing:
    - `PortfolioSummaryDto HypotheticalSummary`
    - `IEnumerable<PortfolioAllocationDto> HypotheticalAllocation`
    - `string HypotheticalSentiment`
    - `string HypotheticalRisk`
    - `string Explanation` (LLM output or deterministic fallback)
    - `decimal ValueDelta`, `decimal ValueDeltaPercentage`
    - `IEnumerable<AllocationDeltaDto> AllocationDeltas`
    - `IEnumerable<AssetWhatIfDeltaDto> PerAssetDeltas`
    - `string RiskDelta`, `string SentimentDelta`

- `AssetWhatIfDeltaDto`
  - Record: `(AssetLabel, BeforePrice, AfterPrice, PriceDelta, PriceDeltaPercentage, BeforeAllocation, AfterAllocation, AllocationDelta, BeforeValue, AfterValue, ValueDelta, ValueDeltaPercentage)`
  - Used to show per-asset impact in the what‑if result.

Other DTOs:
- `AssetBasicInfoDto` — `(Name, Symbol, Type)` returned by portfolio service methods.
- `AllocationDeltaDto` — `(AssetType, BeforePercentage, AfterPercentage, DeltaPercentage)` used in what‑if outputs.

---

## RIA (Frontend) Workflow

How an RIA interacts with the backend:

- View users
  - GET `api/Users` — obtains `UserDto[]` including portfolios and transactions (projection).
- View user portfolios
  - GET `api/Portfolios/users/{userId}` — list portfolios for a user.
- View portfolio summary & allocation
  - GET `api/Portfolios/{id}/summary` — `PortfolioSummaryDto`
  - GET `api/Portfolios/user/{userId}/allocation` — `PortfolioAllocationDto[]`
- View and create transactions
  - GET `api/Transactions/portfolio/{portfolioId}` — transactions list
  - POST `api/Transactions` — create transaction (validated and applied by `TransactionService` which updates portfolio quantities/averages)
- Run What‑If simulations
  - POST `api/AI/whatif` with `WhatIfRequestDto` — returns `WhatIfResultDto` with explanation
- Fetch AI advice
  - GET `api/AI/advice/portfolio/{portfolioId}` — returns `AIAdviceResponseDto`
  - GET `api/AI/advice/transactions/{transactionId}/explain` — request explanation for a particular transaction
- View asset list
  - GET `api/Assets` — list assets (for portfolio creation, symbol display, etc.)

---

## How to Run the Project

Prerequisites
- .NET 8 SDK installed
- MySQL server (compatible with MySQL 8 and the Pomelo provider)
- Optional: PowerShell to run `seed.ps1` for sample data

Configuration
- Connection string: app reads `ConnectionStrings:DefaultConnection` (e.g., `appsettings.Development.json` or environment variable)
- Example `appsettings.json` (snippet)
```json
{
    "ConnectionStrings": {
      "DefaultConnection": "server=localhost;port=3306;database=wealthapp;user=youruser;password=yourpassword"
    },
    "LLM": {
      "UseLLM": false,
      "ApiKey": "",
      "Model": ""
    }
  }
```

Database migrations
- If you use EF migrations:
  - Add migrations: `dotnet ef migrations add InitialCreate -p WealthApp -s WealthApp`
  - Update database: `dotnet ef database update -p WealthApp -s WealthApp`
  - Note: the project registers `UseMySql` with server version `8.0.45-mysql`.

Run the API
- From solution root:
  - `cd WealthApp`
  - `dotnet run`
- Development behavior:
  - Swagger is enabled in Development environment — Swagger UI at `https://localhost:7257/swagger`.
  - CORS policy `AllowFrontend` allows `http://localhost:5173` by default (update in `Program.cs` as needed).

Swagger
- Visit: `https://localhost:7257/swagger` (Dev mode) to explore and test endpoints.

---

## Configuring the LLM

LLM configuration is bound to `WealthApp.Services.LLMSettings` and read from the `LLM` configuration section.

Relevant keys
- `LLM:UseLLM` (bool) — whether to attempt calling the configured LLM.
- `LLM:ApiKey` (string) — API key for the Generative Language API.
- `LLM:Model` (string) — model name (the client normalizes to `models/{Model}`).

Example
```json
"LLM": {
  "UseLLM": true,
  "ApiKey": "<YOUR_API_KEY>",
  "Model": "gemini-pro" // or the model name expected by Google Generative Language API
}
```

Notes
- `GeminiLLMClient` will log an informational message and be effectively disabled if `ApiKey` or `Model` are not set.
- When `LLM:UseLLM` is `false`, the what‑if engine uses a deterministic fallback explanation.
- Debug endpoints:
  - `GET api/AI/whatif/debug/llmsettings` — returns current LLM settings
  - `GET api/AI/whatif/debug/llmtest` — tries a lightweight `ILLMClient.GenerateTextAsync("Ping")` call

Security
- Keep API keys out of source control. Use secrets manager or environment variables for production.

---

## Seeder Guide

A development seeder script is included: `WealthApp/seed.ps1`.

What it does
- Creates sample users (3 users).
- Creates a list of sample assets (common stocks and some crypto).
- Creates portfolios for each user × asset.
- Seeds ~30 days of price history for each asset (POSTs to `api/AssetPrices`).
- Adds a number of `BUY` transactions per portfolio (POSTs to `api/Transactions`).

How to run
1. Ensure the API is running at `https://localhost:7257`.
2. From PowerShell, run:
   - `.\WealthApp\seed.ps1`
3. The script POSTs to the API endpoints and populates the database for development/testing.

Notes
- The seeder assumes the API uses HTTPS at `https://localhost:7257`. Adjust the script `BaseUrl` if running on a different URL/port.
- The seeder creates deterministic but randomized sample data for quick UI development.

---

## Troubleshooting

Common issues and checks

- CORS errors
  - Default CORS policy `AllowFrontend` allows `http://localhost:5173`.
  - If your frontend runs on a different origin, update the origin in `Program.cs` or adjust the CORS policy to allow your frontend.

- Connection string / DB connectivity
  - Ensure `ConnectionStrings:DefaultConnection` is set (appsettings, user secrets or environment variable).
  - Validate MySQL server is running and credentials are correct.
  - Check the `UseMySql(... ServerVersion.Parse("8.0.45-mysql"))` server version matches your server or adjust accordingly.

- Missing or invalid LLM API key
  - If `LLM:ApiKey` or `LLM:Model` are blank, `GeminiLLMClient` logs that it is disabled.
  - `LLM:UseLLM` controls whether the What‑If service attempts LLM calls. If true but client is not configured, requests to LLM endpoints will throw; verify debug endpoints and logs.

- How to verify backend health
  - Visit Swagger UI: `https://localhost:7257/swagger` to exercise endpoints.
  - Use `GET api/AI/whatif/debug/llmtest` to validate LLM connectivity (if enabled).
  - Use `GET api/Users` to ensure DB models and sample data are accessible.

- Errors during migrations
  - Ensure EF Core tools (`dotnet-ef`) installed and working.
  - Ensure you run migrations with the correct project and startup project parameters if solution contains multiple projects.

---

If you need a minimal example of `appsettings.Development.json` or help wiring secrets for `LLM:ApiKey`, or a sample `dotnet ef` command for your environment, tell me which environment (Windows / macOS / Linux) you are using and I will provide the exact commands.
