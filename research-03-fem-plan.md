# FEM 实施计划

## 推荐软件

**ABAQUS/Standard (Implicit)** — ABAQUS is the best choice for this problem for four reasons. (1) Kinematic coupling constraints allow pure bending to be applied cleanly: couple each end cross-section to a reference point, fix one RP, apply a rotation UR3 to the other, leaving U1 free -- this produces constant moment with zero shear force, exactly the required loading. (2) The Python API (odbAccess) enables fully parameterized model generation, batch job submission, and automated post-processing of hundreds of simulations. (3) CPE8 biquadratic plane strain elements capture constant curvature exactly, and path-based extraction of S12 (interface shear) and E11/S11 (through-thickness axial strain/stress) is built-in. (4) The composite/sandwich mechanics community has extensively validated ABAQUS against analytical sandwich beam solutions. A small FEniCSx verification model for 2-3 parameter combinations is recommended as an independent code check.

## 模型几何

Coordinate system: x = axial (origin at mid-span), z = through-thickness (origin at beam mid-plane), y = out-of-plane (unit thickness, plane strain). For N hard layers of thickness t and N-1 soft layers of thickness h: total thickness H_total = N*t + (N-1)*h. Hard layer i occupies z from z_bottom + (i-1)(t+h) to z_bottom + (i-1)(t+h) + t, where z_bottom = -H_total/2. Soft layer j occupies from the top of hard layer j to the bottom of hard layer j+1. Baseline configuration: N=5, t=2 mm, h=5 mm, L=500 mm, H_total=30 mm, aspect ratio L/H=16.7. The analysis zone (where data is extracted) is the central 50-70% of the beam to avoid Saint-Venant end effects, which decay within ~H_total from each end.

## 单元选择

Primary: CPE8 (8-node biquadratic plane strain quadrilateral, 3x3 Gauss integration). Second-order interpolation captures constant curvature exactly and provides accurate stress at integration points. Through-thickness: 8 elements per hard layer (uniform, delta_z = t/8), 10 elements per soft layer (uniform, delta_z = h/10). Axial: uniform mesh in analysis zone with delta_x = 1.0 mm baseline (500 elements for L=500 mm). Element aspect ratio target <= 5 in hard layers. Total ~40,000 CPE8 elements for baseline case (~320,000 nodes). Structured mapped quadrilateral meshing throughout; each layer is a separate rectangular partition. Mesh convergence study at 4 densities (Coarse/Medium/Fine/VeryFine) with acceptance criterion |tau_vfine - tau_fine|/tau_vfine < 1%.

## 材料模型

Both hard and soft layers use linear elastic isotropic constitutive models (ABAQUS *ELASTIC). Hard layers: E_h = 70 GPa (baseline), nu_h = 0.33, t = 2 mm. Soft layers: E_c = 70 MPa (1/1000 of E_h), G_c = 25 MPa (baseline, key sweep parameter), nu_c ~ 0.01 (near-incompressible polymer foam behavior -- foam compresses without lateral expansion, so the key mechanical parameter is G_c, not E_c). Modular ratio eta_E = E_h/E_c ranges from 10^2 to 10^5. Each layer gets its own homogeneous solid section assignment; composite layup capabilities are NOT used since those assume thin-shell kinematics which would mask the through-thickness shear-lag effects.

## 边界条件（纯弯曲）

Pure bending via end-rotation with kinematic coupling: (1) Create reference points RP_left at (-L/2, 0) and RP_right at (+L/2, 0). (2) Couple ALL nodes on each end cross-section (x = +/-L/2, all z) to the corresponding RP using *KINEMATIC COUPLING with DOFs 1-6 constrained. This makes each end face rigid -- a valid Saint-Venant boundary condition; the exact stress state establishes within ~H_total of each end. (3) BC prescription: RP_left: U1=0, U2=0, UR3=0; RP_right: U2=0, UR3=theta_applied, U1 FREE. Theta = kappa_target * L (e.g., 5e-4 rad for kappa=0.001 m^-1, L=0.5 m). Single static step with NLGEOM=OFF (small strains). (4) Stress concentration mitigation: Exclude end regions (|x| > 0.35L) from data extraction. Verify constant curvature in analysis zone by requiring |kappa(x) - kappa_mean|/kappa_mean < 1% for all x in the analysis zone, where kappa(x) is the slope of the least-squares linear fit of epsilon_x(z) at station x. Also verify reaction moment RM at RP_left matches target moment within 1%, and that sigma_z ~ 0 everywhere (max|sigma_z| < 1% of max|sigma_x|).

## 网格设计

Through-thickness: 8 CPE8 elements per hard layer (uniform spacing delta_z = t/8), 10 CPE8 elements per soft layer (uniform spacing delta_z = h/10). Axial: uniform mesh in the central analysis zone (|x| <= 0.35L) with delta_x = 1.0 mm baseline; optional geometric bias (ratio 3:1) outside analysis zone toward ends to reduce element count. Element aspect ratio maintained <= 5 in hard layers. Mesh convergence study: 4 mesh levels (Coarse: 4/5 elements per hard/soft layer, delta_x=2mm ~10k elements; Medium: 6/8, delta_x=1mm ~25k; Fine: 8/10, delta_x=0.5mm ~80k; VeryFine: 12/16, delta_x=0.25mm ~250k). Convergence criterion: relative change in max interface shear stress tau_max at mid-span < 2% between successive levels, < 1% for the final two. Select the coarsest mesh meeting the 2% criterion for production runs. Structured mapped quadrilateral meshing only -- no triangles.

## 参数扫描

- G_c 范围: G_c: 15 log-spaced values from 0.1 to 5000 MPa, covering k = L/l_0 from ~0.005 to ~100, where l_0 = sqrt(E_h*t*h/(2*G_c)). Compute G_c(k) = (E_h*t*h*k^2)/(2*L^2) to anchor sweep to specific k targets (0.1, 1, 10, 100).

- G_c: 15 values (log-spaced, 0.1-5000 MPa)
- t: 3 values (0.5, 2, 10 mm)
- h: 3 values (1, 5, 20 mm)
- E_h: 2 values (35, 210 GPa)
- L: 4 values (100, 200, 500, 2000 mm)
- N: 3 values (3, 5, 9)
- Full factorial: 15*3*3*2*4*3 = 3240 simulations
- Reduced using Latin Hypercube or orthogonal sampling in (log10(k), N) space: ~200-300 core simulations plus ~50 convergence verification runs
- Recommended: ~400 total simulations

### 总计: ~400 个模拟

## 后处理流程

Python pipeline using ABAQUS odbAccess API: (1) extract_interface_shear() -- for each interface i=1 to N-1, extract S12 along a path at constant z=z_interface across the analysis x-range; verify tau ~ 0 at free surfaces (i=0, N). (2) extract_thru_thickness() -- at 51 equally spaced x-stations in the analysis zone, probe E11 and S11 along a vertical path (200 points from z_bottom to z_top). (3) compute_metrics: fit_curvature() performs weighted least-squares linear fit epsilon = epsilon_0 + kappa*z at each x-station to extract local curvature kappa(x); decompose_shear() splits tau into symmetric and antisymmetric components about the mid-plane; compute_plate_index() calculates P = U_bend/U_total where U_bend is the strain energy of the z-proportional part of the strain field; compute_curvature_ratio() gives R_kappa = kappa_local/kappa_beam. (4) batch_run.py: reads parameter matrix CSV, loops over .odb files, calls extraction and computation, outputs consolidated CSV/HDF5 results matrix. (5) Plotting scripts: plot_phase_diagram.py, plot_convergence.py, plot_validation.py. Language: Python 3.10+ with numpy, scipy, matplotlib. ODB extraction runs under 'abaqus python'; plotting runs under standard Python.

## 验证策略

Eight-point automated validation checklist for each simulation: (1) Stress-free surfaces: max|tau(z=+/-H/2)| < 0.1% of max|tau_interior|. (2) sigma_z ~ 0 everywhere: max|sigma_z| < 1% of max|sigma_x|. (3) Constant curvature in analysis zone: R^2 of linear epsilon(z) fit > 0.999 at all x-stations. (4) Antisymmetric sigma_x: |sigma_x(z)+sigma_x(-z)|/|sigma_x|_max < 1%. (5) Reaction moment matches applied: RM_left/M_target within 0.99-1.01. (6) Energy balance: ALLIE/ALLWK within 0.99-1.01. (7) Mesh convergence: relative tau change from coarser mesh < 2%. (8) Stiffness symmetry under reversed loading: UR3 response symmetric within 1%. Analytical benchmark: Fourier series elasticity solution for multi-layer strip under end moments (Lekhnitskii/Airy stress function formalism) compared against FEM results. Target relative L^2 error < 2% in analysis zone. Asymptotic checks: (a) k->0 (membrane): all hard layers must have identical epsilon_x at their centroids (within 1%); (b) k->inf (plate): interface shear -> 0, each layer curvature approaches M_layer/(E_h*I_layer); (c) N->inf: effective bending stiffness approaches homogenized Cosserat continuum prediction.

## 相图构建

Phase diagram axes: x = log10(k) from -2 to +2 (k=0.01 to 100), y = N = 3,5,9. Each (k,N) point colored by the Plate-ness Index P = U_bend/U_total using a perceptually-uniform diverging colormap (blue = membrane P<0.1, white/gray = transition P~0.5, red/orange = plate P>0.9). Three regimes defined: Membrane (P<0.1, layers fully coupled, kappa_i ~ kappa_beam for all i), Transition (0.1<=P<=0.9, partial decoupling), Plate (P>0.9, layers bend independently, tau at interfaces -> 0). Alternative classification via normalized interface shear tau*_max. Phase boundary curves: P=0.1 and P=0.9 contours (dashed). For each N, find k where P=0.5 and fit k_transition(N) ~ C*N^alpha to identify scaling. Three sub-panels show representative through-thickness sigma_x(z) and epsilon_x(z) profiles at one (k,N) point from each regime. Overlay: data point markers, analytical transition line from Fourier solution, error bars from mesh discretization uncertainty.
