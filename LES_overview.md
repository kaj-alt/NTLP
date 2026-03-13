### Purpose of `les.F` in NTLP

- **Main role**: core LES/DNS solver.
- **Responsibilities**:
  - Initialize case-specific base states and fields.
  - Advance prognostic fields with RK3 time stepping.
  - Build right-hand sides for momentum and scalars.
  - Apply advection (skew-symmetric/monotone), SGS diffusion, and source terms.
  - Accumulate horizontal statistics used for output.

### Prognostic fields updated by `les.F`
- **Velocities**: `u, v, w`
- **Turbulent kinetic energy (LES)**: `e`
- **Scalars (array `t(ix,iy,l,iz)`)**:
  - `l=1`: potential temperature (θ)
  - `l=2`: water vapor mixing ratio (qv)
- Scalar tendencies are formed in `r4` and applied each RK substep: `t = t + dtzeta * r4`.

### Key routines inside `les.F`
- **`comp1`**: RK3 time integrator orchestrating one step (calls RHS builders, updates fields, gathers stats).
- **`rhs_uvw` / `rhs_uvw_DNS`**: momentum/TKE right-hand sides (LES vs DNS variants).
- **`rhs_scl`**: scalar right-hand sides (advection, SGS diffusion, vertical flux scheme, sources/BCs).
- Horizontal statistics accumulation (e.g., `uxym`, `txym`, `RHxym`) for diagnostics.

### How `les.F` interacts with other files
- **`defs.F`**
  - Uses modules: `pars`, `fields`, `con_data`, `con_stats`.
  - Provides global arrays (`u,v,w,e,t,r4,...`), base-state profiles, constants/time-step factors, and stats containers.
- **`fft.f`**
  - Spectral derivative utilities for x/y directions and related transforms used within RHS calculations.
- **`particles.f90`**
  - Optional two-way coupling: particle source terms (`partTsrc`, `partHsrc`) can affect scalar budgets/tendencies.
- **`netcdf_io.f90`**
  - Uses horizontally averaged diagnostics produced in `les.F` (e.g., `RHxym`, `txym`, fluxes) and writes them to NetCDF.
- **`measurement.f90`**
  - Optional timing/phase measurement hooks around solver phases.
- **`tec_io.f90`** (if enabled)
  - May export fields updated by `les.F` in Tecplot-compatible formats.

### Notes
- Relative humidity (RH) is a diagnostic computed from `t(:,:,1,iz)` (θ), `t(:,:,2,iz)` (qv), and the base pressure; it is not stored in `t`.
- The vertical transport of scalars can use a monotone upwind option or a skew-symmetric form depending on run settings.

