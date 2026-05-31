# Mapeig d'estats — migració Access → Supabase

## Factures emeses (t_IN → factures)

Camp Access: `in_EstatCob` (BYTE)

| Valor Access | Significat | Estat Supabase |
|---|---|---|
| **1** | Cobrada | `COBRAT` |
| **2** | Pendent de cobrar | `PENDENT` |
| **3** | Rectificativa | `RECTIFICATIVA` |

> ⚠️ IMPORTANT: El mapeig original estava invertit (1→PENDENT, 2→COBRAT).
> El correcte és 1→COBRAT, 2→PENDENT.
> Es va corregir manualment el 2026-05-31 via PATCH a l'API.

### SQL de correcció (si cal tornar a aplicar):
```sql
UPDATE factures SET estat='_TEMP'        WHERE estat='COBRAT';
UPDATE factures SET estat='COBRAT'       WHERE estat='PENDENT';
UPDATE factures SET estat='PENDENT'      WHERE estat='_TEMP';
```

## Factures rebudes (t_INsp → factures_rebudes)

Camp Access: `insp_Comptab` (BOOLEAN)

| Valor Access | Significat | Estat Supabase |
|---|---|---|
| `True` | Comptabilitzada/Pagada | `PAGAT` |
| `False` / NULL | Pendent | `PENDENT` |

Les factures amb `data_termini < CURRENT_DATE` i estat `PENDENT` es marquen com a `PAGAT`
(historials que ja van ser pagades en el seu moment).

### SQL de correcció factures rebudes (si cal):
```sql
UPDATE factures_rebudes
SET estat='PAGAT'
WHERE estat='PENDENT'
AND data_termini < CURRENT_DATE
AND no_factura LIKE 'INSP-%';
```
