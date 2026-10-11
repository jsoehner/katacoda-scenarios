# Interactive Katacoda Scenarios

[![Git](https://app.soluble.cloud/api/v1/public/badges/42b4648b-dc6f-48fe-abba-adade6ca84b0.svg?orgId=320408544746)](https://app.soluble.cloud/repos/details/github.com/jsoehner/katacoda-scenarios?orgId=320408544746)  

[![](http://shields.katacoda.com/katacoda/jsoehner/count.svg)](https://www.katacoda.com/jsoehner "Get your profile on Katacoda.com")

Visit https://www.katacoda.com/jsoehner to view the profile and interactive scenarios

### Writing Scenarios
Visit https://www.katacoda.com/docs to learn more about creating Katacoda scenarios

For examples, visit https://github.com/katacoda/scenario-example

---

## Automated SBOM & CBOM Supply Chain Pipeline

`katacoda-scenarios` integrates an automated Software Bill of Materials (SBOM) and Cryptographic Bill of Materials (CBOM) governance pipeline to satisfy Executive Order 14028, NIST SP 800-218 (SSDF), NIST SP 800-161 (C-SCRM), NTIA Minimum Elements, and federal Post-Quantum Cryptography (PQC) transition mandates (NIST FIPS 203/204/205, CNSA 2.0).

For detailed architectural decisions, trade-offs, and compliance mappings, refer to [ADR-0001: Automated SBOM & CBOM Supply Chain Governance](docs/adr/0001-automated-sbom-cbom-supply-chain-governance.md). See also the latest [Cryptographic Audit & PQC Migration Report](docs/security/CRYPTOGRAPHIC_AUDIT.md).

### CI/CD Workflow Stages (`.github/workflows/sbom-cbom.yml`)

The GitHub Actions workflow triggers on push to `main`/`master`, semantic tags (`v*`), pull requests, and manual triggers (`workflow_dispatch`). It executes across 7 discrete stages:

1. **Dual-Format SBOM Generation**: Employs Anchore/Syft to produce dual-standard machine-readable inventories in CycloneDX 1.6 JSON (`oss/sbom.cyclonedx.json`) and SPDX 2.3 JSON (`oss/sbom.spdx.json`).
2. **CBOM Generation**: Executes `cbomkit-action` to discover cryptographic components, algorithms, and key sizes (`oss/cbom.json`).
3. **Semantic AST Cryptographic Scan & Reconciliation**: Runs `scripts/scan_crypto_ast.py` across source code to detect cryptographic API call sites and reconcile findings directly against the CBOM.
4. **Automated Verification Test Harness**: Executes `scripts/test_boms.sh` to validate schema conformance, non-empty manifests, and minimum component thresholds. Any malformed artifact fails the build.
5. **Post-Quantum Cryptography (PQC) Readiness Assessment**: Analyzes cryptographic algorithms using `scripts/analyze_cbom.py`, evaluating quantum vulnerability vs. post-quantum readiness (CNSA 2.0, FIPS 203/204/205) and rendering a GitHub Actions Step Summary.
6. **Audit Report Publication**: On push to `main`, updates and publishes `docs/security/CRYPTOGRAPHIC_AUDIT.md`.
7. **Artifact Archival**: Uploads all generated BOM artifacts as workflow run artifacts with 90-day retention.

### Local Script Tooling

Developers and DevSecOps engineers can execute the identical BOM toolchain locally:

```bash
# Generate dual-format SBOMs and CBOM locally into oss/
bash scripts/generate_boms.sh -t dir .

# Run semantic AST crypto scanning and CBOM reconciliation
python3 scripts/scan_crypto_ast.py . --cbom oss/cbom.json --output-json oss/crypto-callsites.json

# Execute verification test harness
bash scripts/test_boms.sh oss

# Generate PQC readiness assessment report
python3 scripts/analyze_cbom.py oss/cbom.json \
    --sbom oss/sbom.spdx.json \
    --markdown oss/crypto-summary.md
```
