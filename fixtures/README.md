# Fixtures

- `Bourzma_2025_tracker_and_invoice_register_v9.xlsx`: the 2025 work done by hand (migration A, regression base for matching).
- `bank/swedbank_2025_sample.csv`: Bourzma Swedbank 2025 lines, rebuilt from the parsed statement. It includes the CityBee 30.12.2024 lines and the A-Sound, GD&L, SKR, Bauska and Burka cases. It is a **subset** (76 lines, opening balance only), so use it for parser and matching tests, not for the balance tie-out (C2/C3). **Replace it with a full raw Swedbank export in Phase 2** to confirm the delimiter, encoding and closing-balance records.
- `bank/luminor_sample.csv`: Gabriels adds it in Phase 2 (see the setup checklist).
- `golden/`: Gabriels copies the documents listed in `docs/11-phases-and-tests.md` from Drive in Phase 1.
