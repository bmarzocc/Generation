# Generation

1) Install:

    * scram project CMSSW_9_2_7
    * cd CMSSW_9_2_7/src/
    * cmsenv
    * git clone  git@github.com:bmarzocc/Generation.git
    * cd Generation
    * scram b -j 5

2) Run GEN-SIM production on lxbatch:

    * launch step:
   
      * cd ProduceGENSIM/lxbatch/
      * perl launchJobs_lxbatch_GEN-SIM.pl params_lxbatch_GEN-SIM.CFG
      * sh lancia.sh
      
    * Parameters settings in "params_lxbatch_GEN-SIM.CFG":
   
      * BASEDir: absolute path of the lxbatch directory
      * JOBdir: directory to store job directories
      * JOBCfgTemplate: name of the python to launch
      * HEPMCinput: name and path of the gridpack to be run
      * OUTPUTSAVEPath: path of the output files (EOS directories as default)
      * OUTPUTFILEName: name of the output file.root
      * EVENTSNumber: total number of events to generate
      * EVENTSPerjob: number of events per job
      * EXEName: name of the job executable
      * SCRAM_ARCH: scram_arch
      * X509_USER_PROXY: path of the proxy (before launching better do: export X509_USER_PROXY=/afs/cern.ch/work//XXX/x509up_XXX)
      * QUEUE: lxbatch queue
      
    * NOTE: Any new GEN-SIM step cfg can be adapted in order to work with this package. Change/add the following lines:
    
      ...
      ```Pythonscript
      process.maxEvents = cms.untracked.PSet(
         input = cms.untracked.int32(EVENTSperJOB)
      )
      ```
      ...
      ```Pythonscript
      
      import SimGeneral.Configuration.ThrowAndSetRandomRun as ThrowAndSetRandomRun
      ThrowAndSetRandomRun.throwAndSetRandomRun(process.source,[(JOBID,1)])

      process.RandomNumberGeneratorService = cms.Service("RandomNumberGeneratorService",

         externalLHEProducer = cms.PSet(
            initialSeed = cms.untracked.uint32(SEED1),
            engineName = cms.untracked.string('HepJamesRandom')
         ),
         generator = cms.PSet(
            initialSeed = cms.untracked.uint32(SEED2),
            engineName = cms.untracked.string('HepJamesRandom')
         ),
         VtxSmeared = cms.PSet(
            initialSeed = cms.untracked.uint32(SEED3),
            engineName = cms.untracked.string('HepJamesRandom')
         ),
         g4SimHits = cms.PSet(
            initialSeed = cms.untracked.uint32(SEED4),
            engineName = cms.untracked.string('HepJamesRandom')
         )

      )
      ```
      ...
      ```Pythonscript
      process.RAWSIMoutput = cms.OutputModule("PoolOutputModule",
         SelectEvents = cms.untracked.PSet(
            SelectEvents = cms.vstring('generation_step')
         ),
         compressionAlgorithm = cms.untracked.string('LZMA'),
         compressionLevel = cms.untracked.int32(9),
         dataset = cms.untracked.PSet(
            dataTier = cms.untracked.string('GEN-SIM'),
            filterName = cms.untracked.string('')
         ),
         eventAutoFlushCompressedSize = cms.untracked.int32(20971520),
         fileName = cms.untracked.string('file:OUTPUTFILEName.root'),
         outputCommands = process.RAWSIMEventContent.outputCommands,
         splitLevel = cms.untracked.int32(0)
      )
      ```
      
3)

4) Run GEN analysis:

    * cd Generation/GenStudies
    * GenParticlesAnalysis python/GenParticlesAnalysis_cfg.py
