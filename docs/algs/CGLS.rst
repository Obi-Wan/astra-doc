CGLS
====

This is a CPU implementation of the Conjugate Gradient Least Squares (CGLS) algorithm for 2D data sets. It takes projection data and an initial reconstruction as input, and returns the reconstruction after a specified number of CGLS iterations.

The internal state of the CGLS algorithm is **NOT** reset between astra_mex_algorithm('iterate') calls. Updating the contents of the projection data and reconstruction data astra_mex_data2d objects between astra_mex_algorithm('iterate') calls will produce undefined behaviour.

Supported geometries: parallel, fanflat, fanflat_vec, matrix.

Configuration options
---------------------

=============================== ========	=======================================
name 				type 		description
=============================== ========	=======================================
cfg.ProjectorId 		required 	The astra_mex_projector ID of the projector.
cfg.ProjectionDataId 		required 	The astra_mex_data2d ID of the projection data
cfg.ReconstructionDataId 	required 	The astra_mex_data2d ID of the reconstruction data. The content of this when starting CGLS is used as the initial reconstruction.
cfg.option.SinogramMaskId 	optional 	If specified, the astra_mex_data2d ID of a projection-data-sized volume to be used as a mask.
cfg.option.ReconstructionMaskId optional 	If specified, the astra_mex_data2d ID of a volume-data-sized volume to be used as a mask.
=============================== ========	=======================================

Example
-------

.. code-block:: matlab

	%% create phantom
	V_exact = phantom(256);

	%% create geometries and projector
	proj_geom = astra_create_proj_geom('parallel', 1.0, 256, linspace2(0,pi,180));
	vol_geom = astra_create_vol_geom(256,256);
	proj_id = astra_create_projector('linear', proj_geom, vol_geom);

	%% create forward projection
	[sinogram_id, sinogram] = astra_create_sino(V_exact, proj_id);

	%% reconstruct
	recon_id = astra_mex_data2d('create', '-vol', vol_geom, 0);
	cfg = astra_struct('CGLS');
	cfg.ProjectorId = proj_id;
	cfg.ProjectionDataId = sinogram_id;
	cfg.ReconstructionDataId = recon_id;
	cgls_id = astra_mex_algorithm('create', cfg);
	astra_mex_algorithm('iterate', cgls_id, 100);
	V = astra_mex_data2d('get', recon_id);
	imshow(V, []);

	%% garbage disposal
	astra_mex_data2d('delete', sinogram_id, recon_id);
	astra_mex_projector('delete', proj_id);
	astra_mex_algorithm('delete', cgls_id);


