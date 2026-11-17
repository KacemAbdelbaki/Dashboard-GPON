# Dashboard GPON

An admin dashboard for managing a GPON (fiber-to-the-home) network inventory. It tracks the equipment along each
fiber chain, from the OLT in the central office through hubs and sub-boxes to the end box, and shows it on a map.
The interface is in French.

Built with Laravel 11, Blade and [Leaflet](https://leafletjs.com).

> **Status:** work in progress. The chain map (`/chaine/map`) is still being built.

## Features

- **Equipment management (CRUD)** for each level of the network:
  - **OLT** (Optical Line Terminal): model, type, address, optical central office, slot/board number, port
    capacity, commissioning date
  - **Cartes** (line cards): model, number of ports, slot
  - **Hubs**: model, serial number, assigned ports, number of active chains, parent OLT
  - **Sub-boxes / end boxes**: model, serial number, position in the chain, previous box, parent hub
- **Map view**: every piece of equipment has GPS coordinates and is shown on a Leaflet map.
- **Chain view**: the dashboard rebuilds each chain (OLT → hub → sub-boxes → end box) and lets you search by OLT
  name.

## Network model

```
OLT ──< Carte          (olt_carte)
 │
 └── Hub ──< Carte     (hub_carte)
      │
      └──< SubBox → SubBox → … → End box
```

| Table | Model |
|-------|-------|
| `olt` | `App\Models\OLT` |
| `carte` | `App\Models\Carte` |
| `hub` | `App\Models\Hub` |
| `sub_box` | `App\Models\SubBox` |

Coordinates are stored in a spatial `coordonne` column and read with `ST_X` / `ST_Y`, so the app needs **MySQL**
(SQLite does not support these functions).

## Getting started

### Requirements

- PHP 8.2+ and [Composer](https://getcomposer.org)
- MySQL 8+

### 1. Install

```bash
git clone https://github.com/KacemAbdelbaki/Dashboard-GPON.git
cd Dashboard-GPON
composer install
cp .env.example .env
php artisan key:generate
```

### 2. Configure the database

In `.env`, switch from SQLite to MySQL:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=dashboard_gpon
DB_USERNAME=root
DB_PASSWORD=
```

> **Note:** the migrations for the equipment tables are not in the repository yet. Create the `olt`, `carte`,
> `hub`, `sub_box`, `olt_carte` and `hub_carte` tables yourself, using the columns listed in each model's
> `$fillable`. The `coordonne` column must be a spatial `POINT`.

### 3. Run

```bash
php artisan serve
```

Open http://localhost:8000.

## Pages

| Path | Page |
|------|------|
| `/` | Dashboard: chains and map, searchable by OLT name |
| `/chaine/map` | Chain map (in progress) |
| `/olts`, `/ajouterOLT`, `/modifierOLT/{id}` | OLTs |
| `/cartes`, `/ajouterCarte`, `/modifierCarte/{id}` | Line cards |
| `/hubs`, `/ajouterHub`, `/modifierHub/{id}` | Hubs |
| `/subBoxs`, `/ajouterSubBox`, `/modifierSubBox/{id}` | Sub-boxes and end boxes |

## Project structure

```
app/
├── Http/Controllers/Admin/   # Dashboard, OLT, Carte, Hub, SubBox
└── Models/                   # OLT, Carte, Hub, SubBox
resources/views/Admin/        # One folder per equipment type, plus layout and map partials
routes/web.php
public/                       # Admin template assets (CSS, JS, fonts)
```

## Related

[GPON-Project---Huawei](https://github.com/KacemAbdelbaki/GPON-Project---Huawei) is a later GPON management
project, built with Spring Boot microservices and Angular.
