# Guida
# 📋 Guida Completa — Esercizio Autovelox

> SQL Server + Entity Framework Core + ASP.NET WebAPI

---

## Indice

- [Parte 1 — SQL Server](#parte-1--sql-server)
  - [STEP 1 · Ripristino del Database](#step-1--ripristino-del-database)
  - [STEP 2 · Creare le Relazioni tra le Tabelle](#step-2--creare-le-relazioni-tra-le-tabelle)
  - [STEP 3 · Creare l'Account swd2426](#step-3--creare-laccount-swd2426)
  - [STEP 4 · Verificare l'Account — 100 Autovelox del Piemonte](#step-4--verificare-laccount--100-autovelox-del-piemonte)
  - [STEP 5 · Correggere Latitudine e Longitudine](#step-5--correggere-latitudine-e-longitudine)
  - [STEP 6 · Autovelox sul 45° Parallelo](#step-6--autovelox-sul-45-parallelo)
  - [STEP 7 · Query Richieste](#step-7--query-richieste)
- [Parte 2 — Visual Studio / .NET](#parte-2--visual-studio--net)
  - [STEP 8 · Creare la Soluzione](#step-8--creare-la-soluzione)
  - [STEP 9 · Aggiungere il Progetto Class Library](#step-9--aggiungere-il-progetto-class-library)
  - [STEP 10 · Installare Entity Framework Core](#step-10--installare-entity-framework-core)
  - [STEP 11 · Scaffold del Database](#step-11--scaffold-del-database-database-first)
  - [STEP 12 · Aggiungere il Progetto WebAPI](#step-12--aggiungere-il-progetto-webapi)
  - [STEP 13 · Aggiungere il Riferimento a Autovelox.Data](#step-13--aggiungere-il-riferimento-a-autoveloxdata)
  - [STEP 14 · Configurare la Connection String](#step-14--configurare-la-connection-string)
  - [STEP 15 · Creare i Controller](#step-15--creare-i-controller)
  - [STEP 16 · Avviare e Testare](#step-16--avviare-e-testare)
- [Riepilogo Endpoint](#riepilogo-endpoint)

---

## PARTE 1 — SQL SERVER

### STEP 1 · Ripristino del Database

1. Scarica il file di backup `.bak` dal link Autovelox sul FAD
2. Apri **SQL Server Management Studio (SSMS)** e connettiti con autenticazione Windows
3. Nel pannello Object Explorer → clic destro su **Databases** → **Restore Database...**
4. Seleziona **Device** → clicca i tre puntini `...` → **Add** → naviga fino al file `.bak` → **OK**
5. Clicca **OK** → attendi il messaggio *"Database restored successfully"*

---

### STEP 2 · Creare le Relazioni tra le Tabelle

Apri una **New Query** sul database Autovelox ed esegui:

```sql
USE Autovelox;
GO

-- Regione -> RipartizioneGeografica
ALTER TABLE Regione
ADD CONSTRAINT FK_Regione_RipartizioneGeografica
FOREIGN KEY (IdRipartizioneGeografica) REFERENCES RipartizioneGeografica(Id);

-- Provincia -> Regione
ALTER TABLE Provincia
ADD CONSTRAINT FK_Provincia_Regione
FOREIGN KEY (IdRegione) REFERENCES Regione(Id);

-- Comune -> Provincia
ALTER TABLE Comune
ADD CONSTRAINT FK_Comune_Provincia
FOREIGN KEY (IdProvincia) REFERENCES Provincia(Id);

-- Mappa -> Comune
ALTER TABLE Mappa
ADD CONSTRAINT FK_Mappa_Comune
FOREIGN KEY (IdComune) REFERENCES Comune(IdComune);
```

Per il **diagramma visivo**: Object Explorer → espandi `Autovelox` → clic destro su **Database Diagrams** → **New Database Diagram** → aggiungi tutte le tabelle.

---

### STEP 3 · Creare l'Account swd2426

Apri una nuova query sul database **master** ed esegui:

```sql
USE master;
GO

CREATE LOGIN swd2426
WITH PASSWORD = 'its-2026',
DEFAULT_DATABASE = master;
GO

USE Autovelox;
GO

CREATE USER swd2426 FOR LOGIN swd2426;
GO

ALTER ROLE db_owner ADD MEMBER swd2426;
GO
```

> ⚠️ Con `db_owner` l'utente può fare tutto su Autovelox ma **non** ha accesso ad altri database.

---

### STEP 4 · Verificare l'Account — 100 Autovelox del Piemonte

Connettiti a SSMS con:
- **Login:** `swd2426`
- **Password:** `its-2026`

Poi esegui (deve restituire **100 record**):

```sql
USE Autovelox;
GO

SELECT m.*
FROM Mappa m
INNER JOIN Comune c ON m.IdComune = c.IdComune
INNER JOIN Provincia p ON c.IdProvincia = p.Id
INNER JOIN Regione r ON p.IdRegione = r.Id
WHERE r.Denominazione = 'Piemonte';
```

---

### STEP 5 · Correggere Latitudine e Longitudine

In Italia la latitudine è ~37–47 e la longitudine è ~6–18. Se invertite, correggi così:

```sql
-- Visualizza record con coordinate sbagliate
SELECT Id, Latitudine, Longitudine
FROM Mappa
WHERE Latitudine < 36 OR Latitudine > 48
   OR Longitudine < 6 OR Longitudine > 19;

-- Correzione: scambia i valori se invertiti
UPDATE Mappa
SET Latitudine = Longitudine,
    Longitudine = Latitudine
WHERE Latitudine < 6 OR Latitudine > 19;
```

---

### STEP 6 · Autovelox sul 45° Parallelo

```sql
SELECT *
FROM Mappa
WHERE FLOOR(Latitudine) = 45;
-- oppure con tolleranza:
-- WHERE Latitudine >= 45.0 AND Latitudine < 46.0
```

---

### STEP 7 · Query Richieste

**Query 1 — Comuni con almeno un autovelox in provincia di Teramo:**

```sql
SELECT DISTINCT c.Denominazione AS Comune
FROM Mappa m
INNER JOIN Comune c ON m.IdComune = c.IdComune
INNER JOIN Provincia p ON c.IdProvincia = p.Id
WHERE p.Denominazione = 'Teramo'
ORDER BY c.Denominazione;
```

**Query 2 — Anno di installazione e numero di comuni:**

```sql
SELECT
    m.AnnoInserimento AS Anno,
    COUNT(DISTINCT m.IdComune) AS NumeroComuni
FROM Mappa m
GROUP BY m.AnnoInserimento
ORDER BY m.AnnoInserimento;
```

---

## PARTE 2 — VISUAL STUDIO / .NET

### STEP 8 · Creare la Soluzione

1. Apri **Visual Studio** → **Create a new project**
2. Cerca **Blank Solution** → **Next**
3. Nome soluzione: `AutoveloxProject` → **Create**

---

### STEP 9 · Aggiungere il Progetto Class Library

1. Clic destro sulla soluzione → **Add** → **New Project**
2. Cerca **Class Library (.NET)** → **Next**
3. Nome: `Autovelox.Data` — Framework: `.NET 10.0` → **Create**
4. Elimina il file `Class1.cs` creato automaticamente

---

### STEP 10 · Installare Entity Framework Core

Clic destro su `Autovelox.Data` → **Manage NuGet Packages** → installa:

| Pacchetto | Descrizione |
|---|---|
| `Microsoft.EntityFrameworkCore` | EF Core base |
| `Microsoft.EntityFrameworkCore.SqlServer` | Provider SQL Server |
| `Microsoft.EntityFrameworkCore.Tools` | Scaffolding/Migrations |

Oppure da **Package Manager Console** (Tools → NuGet → Package Manager Console):

```powershell
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
```

---

### STEP 11 · Scaffold del Database (Database-First)

Dalla **Package Manager Console**, con progetto default `Autovelox.Data`:

```powershell
Scaffold-DbContext "Server=.\SQLEXPRESS;Database=Autovelox;User Id=swd2426;Password=its-2026;TrustServerCertificate=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Force
```

Genera automaticamente nella cartella `Models/`:

```
Autovelox.Data/
└── Models/
    ├── Mappa.cs
    ├── Comune.cs
    ├── Provincia.cs
    ├── Regione.cs
    ├── RipartizioneGeografica.cs
    └── AutoveloxContext.cs   ← il DbContext
```

---

### STEP 12 · Aggiungere il Progetto WebAPI

1. Clic destro sulla soluzione → **Add** → **New Project**
2. Cerca **ASP.NET Core Web API** → **Next**
3. Nome: `Autovelox.WebAPI` — Framework: `.NET 10.0`
4. Lascia spuntato **Use controllers** → **Create**

---

### STEP 13 · Aggiungere il Riferimento a Autovelox.Data

Clic destro su `Autovelox.WebAPI` → **Add** → **Project Reference** → spunta `Autovelox.Data` → **OK**

---

### STEP 14 · Configurare la Connection String

In `appsettings.json` del WebAPI:

```json
{
  "ConnectionStrings": {
    "AutoveloxDB": "Server=.\\SQLEXPRESS;Database=Autovelox;User Id=swd2426;Password=its-2026;TrustServerCertificate=True;"
  }
}
```

In `Program.cs`:

```csharp
using Autovelox.Data.Models;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AutoveloxContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("AutoveloxDB")));

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
app.UseSwagger();
app.UseSwaggerUI();
app.MapControllers();
app.Run();
```

---

### STEP 15 · Creare i Controller

#### MappeController.cs

```csharp
[ApiController]
[Route("api/[controller]")]
public class MappeController : ControllerBase
{
    private readonly AutoveloxContext _context;
    public MappeController(AutoveloxContext context) => _context = context;

    // GET: api/Mappe — tutti gli autovelox
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var mappe = await _context.Mappas
            .Include(m => m.IdComuneNavigation)
            .ToListAsync();
        return Ok(mappe);
    }

    // GET: api/Mappe/5 — dettaglio singolo completo
    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        var mappa = await _context.Mappas
            .Include(m => m.IdComuneNavigation)
                .ThenInclude(c => c.IdProvinciaNavigation)
                    .ThenInclude(p => p.IdRegioneNavigation)
                        .ThenInclude(r => r.IdRipartizioneGeograficaNavigation)
            .FirstOrDefaultAsync(m => m.Id == id);
        if (mappa == null) return NotFound();
        return Ok(mappa);
    }

    // GET: api/Mappe/search?regione=Piemonte&provincia=Torino&comune=Torino
    [HttpGet("search")]
    public async Task<IActionResult> Search(
        [FromQuery] string? regione,
        [FromQuery] string? provincia,
        [FromQuery] string? comune)
    {
        var query = _context.Mappas
            .Include(m => m.IdComuneNavigation)
                .ThenInclude(c => c.IdProvinciaNavigation)
                    .ThenInclude(p => p.IdRegioneNavigation)
            .AsQueryable();

        if (!string.IsNullOrEmpty(regione))
            query = query.Where(m =>
                m.IdComuneNavigation.IdProvinciaNavigation
                  .IdRegioneNavigation.Denominazione == regione);
        if (!string.IsNullOrEmpty(provincia))
            query = query.Where(m =>
                m.IdComuneNavigation.IdProvinciaNavigation.Denominazione == provincia);
        if (!string.IsNullOrEmpty(comune))
            query = query.Where(m =>
                m.IdComuneNavigation.Denominazione == comune);

        return Ok(await query.ToListAsync());
    }
}
```

#### RipartizioniGeograficheController.cs

```csharp
[ApiController]
[Route("api/[controller]")]
public class RipartizioniGeograficheController : ControllerBase
{
    private readonly AutoveloxContext _context;
    public RipartizioniGeograficheController(AutoveloxContext ctx) => _context = ctx;

    [HttpGet]
    public async Task<IActionResult> GetAll() =>
        Ok(await _context.RipartizioneGeograficas.ToListAsync());

    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id) =>
        Ok(await _context.RipartizioneGeograficas.FindAsync(id));
}
```

#### RegioniController.cs

```csharp
[ApiController]
[Route("api/[controller]")]
public class RegioniController : ControllerBase
{
    private readonly AutoveloxContext _context;
    public RegioniController(AutoveloxContext ctx) => _context = ctx;

    [HttpGet]
    public async Task<IActionResult> GetAll() =>
        Ok(await _context.Regiones.ToListAsync());

    // GET: api/Regioni/search?ripartizione=Nord-Ovest
    [HttpGet("search")]
    public async Task<IActionResult> Search([FromQuery] string? ripartizione)
    {
        var query = _context.Regiones
            .Include(r => r.IdRipartizioneGeograficaNavigation)
            .AsQueryable();
        if (!string.IsNullOrEmpty(ripartizione))
            query = query.Where(r =>
                r.IdRipartizioneGeograficaNavigation.Denominazione == ripartizione);
        return Ok(await query.ToListAsync());
    }
}
```

#### ProvinceController.cs

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProvinceController : ControllerBase
{
    private readonly AutoveloxContext _context;
    public ProvinceController(AutoveloxContext ctx) => _context = ctx;

    [HttpGet]
    public async Task<IActionResult> GetAll() =>
        Ok(await _context.Provincias.ToListAsync());

    // GET: api/Province/search?regione=Piemonte
    [HttpGet("search")]
    public async Task<IActionResult> Search([FromQuery] string? regione)
    {
        var query = _context.Provincias
            .Include(p => p.IdRegioneNavigation)
            .AsQueryable();
        if (!string.IsNullOrEmpty(regione))
            query = query.Where(p =>
                p.IdRegioneNavigation.Denominazione == regione);
        return Ok(await query.ToListAsync());
    }
}
```

#### ComuniController.cs

```csharp
[ApiController]
[Route("api/[controller]")]
public class ComuniController : ControllerBase
{
    private readonly AutoveloxContext _context;
    public ComuniController(AutoveloxContext ctx) => _context = ctx;

    [HttpGet]
    public async Task<IActionResult> GetAll() =>
        Ok(await _context.Comunes.ToListAsync());

    // GET: api/Comuni/search?provincia=Torino
    [HttpGet("search")]
    public async Task<IActionResult> Search([FromQuery] string? provincia)
    {
        var query = _context.Comunes
            .Include(c => c.IdProvinciaNavigation)
            .AsQueryable();
        if (!string.IsNullOrEmpty(provincia))
            query = query.Where(c =>
                c.IdProvinciaNavigation.Denominazione == provincia);
        return Ok(await query.ToListAsync());
    }
}
```

---

### STEP 16 · Avviare e Testare

1. Premi **F5** per avviare il WebAPI
2. Si apre **Swagger UI** nel browser automaticamente
3. Clicca su ogni endpoint → **Try it out** → **Execute** per testare

---

## Riepilogo Endpoint

| Metodo | URL | Descrizione |
|---|---|---|
| `GET` | `/api/Mappe` | Tutti gli autovelox |
| `GET` | `/api/Mappe/{id}` | Dettaglio autovelox completo |
| `GET` | `/api/Mappe/search?regione=X&provincia=Y&comune=Z` | Ricerca autovelox |
| `GET` | `/api/RipartizioniGeografiche` | Tutte le ripartizioni geografiche |
| `GET` | `/api/Regioni` | Tutte le regioni |
| `GET` | `/api/Regioni/search?ripartizione=X` | Regioni per ripartizione |
| `GET` | `/api/Province` | Tutte le province |
| `GET` | `/api/Province/search?regione=X` | Province per regione |
| `GET` | `/api/Comuni` | Tutti i comuni |
| `GET` | `/api/Comuni/search?provincia=X` | Comuni per provincia |

---

*Guida generata per ripasso esame — In bocca al lupo! 🍀*
