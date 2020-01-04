# Joe's ORNL Projects
*and how to maintain them with his ~~demise~~ departure*


Joe's projects/tasks at ORNL can be grouped into three main categories, which 
are briefely described here. 

Use [this repositories issues](https://github.com/jhkennedy/JORNL/issues) 
to ask questions, get clarification, etc. about any 
of these. 

# LIVVkit

LIVVkit, as a project, has three basic parts. verification of Ice Sheet Models 
(ISMs), validation of ISMs, and general software engineering/devOps tasks. 
Right now, LIVVkit is funded through the ProbSLR
project, and so is only *supporting* (ish) the MALI and BISICLES ISMs.  

## Verification

In order to regularly verify MALI and BISICLES, a DevOps infrastructure needs to
be setup to regularly build and test each model. Because this project was originally
targeted as CISM, the then DOE ice sheet model, most of this type of work was done for
CISM and is unfortunately pretty useless for MALI and BISICLES. 

### CISM history

CISM had a series of test that were run with a regression test python script inside
the CISM repo. This was extended a bit by my and a "Build and Test Structure" (BATS)
was developed to run all the tests Bill Lipscomb (primarily; He's now at NCAR) 
was interested in being part of the verification testing. BATS produced a rigid
directory structure containing the metadata (2020 vision: bad choice) of some of 
the model setup (e.g., size of the grid and number of processes used during the run).
When [Kennedy et al. (2017)](https://doi.org/10.1002/2017MS000916) was published,
LIVVkit was analyzing changes between CISM 2.0.0 and 2.0.6 BATS output, which are
still the example output in the LIVVkit docs.  

*Note: When E3SM and CESM split I pulled the existing BATS structure out of CISM and archived 
it here: https://github.com/LIVVkit/BATS*

### Community dis-interest and the catch-22

After [Kennedy et al. (2017)](https://doi.org/10.1002/2017MS000916) was published, 
it became clear that really the only person interested in the verification work was
Bill Lipscomb and after the E3SM-CESM split, no one on the E3SM side was really
interested in *acutally* using LIVVkit or building it into their workflow (no one 
at LANL had ever even *run* LIVVkit at this point). Adding to the general disinterest
was the fact that CISM was no longer the E3SM supported model, all the verification
work that went into CISM and BATS would somehow need to recreated. 

Also, about this time, it was clear from the wider scientific community that no one 
was really interested in building in *their* models, but would totes love it if
*we* built it in for them and it was already ready for them to use. Hence the catch 22:
in order to get a development community, you need to provide the community with something 
working and useful, but to get something working and useful you need help from the 
community (this is doubly true for validation, where even working with some of the 
observational data like GRACE requires data experts...) 

Finally, inside the E3SM ISM community, it became increasingly clear that *nobody cares
about verification* and that validation is how you get people excited and build a community. 

And so, the verification work was mostly ignored to focus on validation and getting out
[Evans et al. (2019)](https://doi.org/10.5194/gmd-12-1067-2019). 
  
### Renewed interest and current Work

Though a variety of forces inside and outside the E3SM ISM community, interest in
verification picked back up again in 2019, and now, people are really interested in 
getting some verification going to MALI and BISICLES. So, here's what I suggest needs
to be done:
1. **DevOps:** A "DevOps" pipeline should be built to regularly (ideally continuously) build both
   MALI and BISICLES and then run them through some regression tests
2. **LIVVkit:** LIVVkit needs to be able to ingest the regression test data from both 
   MALI and BISCLES and analyze them
   
Both (1) and (2) need to report the status of the build, test, and analyses somewhere
that can be quickly viewed and analyzed by developers. 

#### 1. DevOps

Because BATS was really develop for and inside of CISM it's not particularly flexible 
and a lot of work would need to be done to use it for either MALI or BISCLES. Additionally, 
the build and test running and reporting was handled by a 
[custom pipeline I rolled](https://github.com/jhkennedy/livv-nightly), which is
not particularly great, different than other verification/test reporting done in 
the E3SM community, and doesn't take advantage of any of the modern industry tools
developed specifically to do DevOps well. 

So, with 2-years more experience in DevOps and a somewhat forced break from 
previous methods, we were given the oppertunity to start anew. 

Some identified requirements:
1. To run at least nightly tests, but would love some sort of actual continuous testing  
2. To use CDASH as the build and test reporting requirements as that's what the
  rest of E3SM is using
3. Automatically publish the LIVVkit analyses websites somewhere for viewing (and
  include that in the CDASH report -- a easy to copy URL is fine, but ideally a
  hyperlink)
4. Be able to run both Greenland and Antarctica scale simulation **and** measure their performance

Also, pretty much the only machine the E3SM ISM community is currently interested in
is Cori at NERSC, so we only need to target that. BUT, that's unlikely to be true
in the long run, so ideally deploying on other machines should be easy.

I started work towards this end in https://github.com/LIVVkit/dashboard

##### LIVVkit's Dashboard

So, because we want to report to CDASH, but neither MALI or BISICLES is using
CTest to run their tests (nor really using CMake to build...), we can't easily 
use CTest to just run and report to CDASH as intended. This is true throughout
E3SM community, and various groups have worked out different ways to generate an 
appropriate XML file to push to CDASH. 

E3SM, via CIME (I think), manually creates
their own XML files and then sends them to CDASH (Jim Foucar built them, I believe).
Likewise, Irina Tezaur at Sandia also rolls her own XML and sends it off to CDASH
for her various projects. I looked at both their methods and considered also doing
the same, but, long term maintenance wise, that sounded fragile and like a big
headache. Meanwhile I found [pyctest](https://pyctest.readthedocs.io/en/latest/)
which generates and submits the CDASH XML for you, and sounded like a possibly
better option. Additionly, both Jim and Irina were curious how well it works and 
this seemed like a good option to give it ago. 

So, the basic premise with `dashboard` is that you can write a YAML profile which
`worker.py` will use to build and run the specified model via `pyctest`. See the 
`dashboard` readme for usage information. Right now, both MALI and BISICLES will build
and report that status to the [LIVVkit CDASH board](https://my.cdash.org/index.php?project=LIVVkit). 

###### Left to do:
* **Set up nightly testing:** There was ongoing work to setup a GitLab extension so that repositories could be
  mirrored at [DOE Code](https://gitlab.osti.gov) and use GitLabs CI/CD tools to 
  run tests on any DOE supported machine. At OLCF via code.ornl.gov, you can do that
  for OLCF machines (they started this work), but it's not currently worth doing much
  with as the person that was working on it left and it doesn't extend past OLCF right
  now. 
  
  So, for the NERSC machines we care about, they allow users to setup cron jobs. 
  A cron job should be set up for each nersc profile in `dasboard` -- right now,
  it's just `build_mali_cori.yaml` and `build_bisicles.yaml`.
  
* **Verify the build:** The above two profiles currently just simply build the model
  and don't do much else. MALI doesn't verify the build in any way (see the test script),
  and BISICLES actually tries to run a simple "hello world" type test, but it *actually*
  fails with a liking error. I *think* because it should be run on an interactive/compute node, 
  not a login node. 
  
  But, building is a big waste of allocation time, so you don't want to build on a compute
  node. And since pyctest really doesn't understand batch queues or whatever, you have to
  go though all the work phases for each run. So, You could just keep the build
   dumb and on the login node -- just check if the 
  executable exists, and then setup a profile for the actual tests that the "build" command is
  just a simple check to see if the executable exists (and probably is not an old one, somehow)
  
  All of that will need to be coordinated via cron somehow -- you could maybe make the simple "test"
  command, instead of verifying the executable exists, just submit a batch job to actually run
  the tests profile...
  
* **Real regression testing:** MALI might have regression tests vial the MPAS testing
  thing called COMPASS and BISICLES has a bunch of tests in it as well, but there should
  be a real sit-down-think-it-out session with LANL on what regression tests each model
  should have/run. It's all rather haphazard right now. 
  
* **LIVVkit ingest:** After all the above is figured out, then LIVVkit will have to ingest
  all the run data and do the analyses. Once LIVVkit can injest them, you can have it run (via cron?)
  after all the regression test are done running, and then write the output website to 
  one of NERSCs hosted directories (either user or project) so it's emediately viewable.
  
* **Report notification:** I believe, once things are reporting to CDASH, you can set it up
  to send emails to interested parties if things fail or not (or daily digests, even)

## Verification

## SE/DevOps

This section is just a collection of some need-to-know Software Enginnering and DevOps things,
like how to release a new version of LIVVkit. 

### LIVVkit Release Steps

1. Open a PR from `develop` to `master` and ask for a least one reviewer. You'll want to 
   write a detailed discussion of the changes in this release (and why) which you'll likely 
   use for both the merge commit message and the release notes. Then check:
   * all tests pass
   * you've bumped the version number appropriately
   
1. Test LEX and EVV with this new release. If either break, this should be a major release!

1. Merge the release using the PR discussion to write an informative merge commit message 
   (look back at previous release commits for an example)

1. publish the release on Github -- again use the PR discussion to write an informative 
   release note (look back at previous release notes for examples)

1. Publish to PyPI (the python packaging index)
   * For reference, most of the steps are outlined here: https://packaging.python.org/tutorials/packaging-projects/
   * Add a `.pypirc` to your home directory that looks like:
     ```ini
     [distutils]
     index-servers =
       pypi
       test
    
     [pypi]
     username=jhkennedy
    
     [test]
     repository=https://test.pypi.org/legacy/
     username=jhkennedy
     ```
     where you've replace my username with yours.
      
   * Update your environment with the needed packaging tools: 
     ```bash
     pip install --user --upgrade setuptools wheel twine
     ```
   * From the top of the LIVVkit repo, build your package: 
     ```bash
     git fetch --all
     git checkout master
     git pull --ff-only
     python setup.py sdist bdist_wheel
     ```
   * Upload to the test PyPI site
     ```bash
     twine upload -r test dist/*
     ```
     and check that everything looks good on the package site (URL will be printed by twine).
   * Create a new env and test the install of LIVVkit from the test PyPI site
     ```bash
     conda env create -n livv-3.0.0-test -f LIVVkit.yml
     conda activate livv-3.0.0-test
     pip install --index-url https://test.pypi.org/simple/ --no-deps livvkit
     livv -v tests/_data/cism-2.0.0-tests/titan-gnu/CISM_glissade \
             tests/_data/cism-2.0.6-tests/titan-gnu/CISM_glissade \
             -o tests/vv_test -s -p 3
     ```
   * When everything looks good, upload to the production PyPI site:
     ```bash
     twine upload -r pypi dist/*
     ```
1. Update the example website. Since you (should have) just ran a verification test, we'll
   use that to update the example website. The example website lives in the `gh-pages` branch, 
   but, because it's an orphaned branch it's best to clone just that branch.      
   
   So, lets say the LIVVkit repo is located at `${LIVVKIT}` and you want the `gh-pages` clone to 
   be at `${LIVVKIT_EXAMPLE}`:
   ```bash
   git clone --single-branch --branch gh-pages git@github.com:LIVVkit/LIVVkit.git ${LIVVKIT_EXAMPLE}
   cd ${LIVVKIT_EXAMPLE}
   # blow away the old website
   rm -r ./*  # CAREFUL
   # get the new example website
   cp -r ${LIVVKIT}/tests/vv_test/* .
   git add .
   # check it looks okay
   git status
   # looks good, so
   git commit -m "LIVVkit 3.0.0 Example"
   git push
   ```
   Now, after a few minutes, you should see the new website up at http://livvkit.github.io/LIVVkit/
   
1. Update the docs. Because we used the `gh-pages` branch for the example website, we host the
   at https://livvkit.github.io/Docs/, and updating this will be similar to the example website.
   First, let's build the docs:
   ```bash
   cd ${LIVVKIT}
   # Make sure the livv-3.0.0-test env we created earlier is active
   conda activate livv-3.0.0-test
   # uninstall our version from the test pypi site
   pip uninstall livvkit
   # install a develop/editable version
   pip install -e .[develop]
   # now, the docs!
   cd docs
   ./generate_docs.sh
   ``` 
   Now, we need to grab the `gh-pages` branch from the https://github.com/LIVVkit/Docs repo
   -- we'll place it in `${LIVVKIT_DOCS}` -- and then update the docs. 
   ```bash
   git clone --single-branch --branch gh-pages git@github.com:LIVVkit/Docs.git ${LIVVKIT_DOCS}
   cd ${LIVVKIT_DOCS}
   # blow away the old docs
   rm -r ./*  # CAREFUL
   # get the new example website
   cp -r ${LIVVKIT}/docs/_build/html/* .
   git add .
   # check it looks okay
   git status
   # looks good, so
   git commit -m "LIVVkit 3.0.0 Docs"
   git push
   ```
   Now, after a few minutes, you should see the new docs up at http://livvkit.github.io/Docs/
   
   *Note: it might be worth switching to https://readthedocs.org/ or similar...*
   
1. Merge master into dev and bump version number in `develop` to next target
   ```bash
   git checkout develop
   git merge --ff-only master
   # Edit version number to next target, e.g.: __version_info__ = (3, 1, 0)
   vim livvkit/__init__.py
   git add livvkit/__init__.py
   git commit -m "Bump version to next target: v3.1.0"
   git push
   ```
1. Update the conda-forge LIVVkit feedstock
   * Once the new released is pushed to PyPI, conda-forge's `regro-cf-autotick-bot` should 
     see the new release and open a PR to update the recipe, e.g.: 
         https://github.com/conda-forge/livvkit-feedstock/pull/2
     it usually only takes a few minutes, but may take up to ~24 hours. If you're 
     listed as a package maintainer on conda-forge for LIVVkit, you'll get an 
     email when it happens.
   * Once it's open, determine if changes need to be made. If so:
     * clone the bot's fork of the livvkit-feedstock, and check out the branch 
       (will say what branch and the fork location at the bottom of the PR!):
       ```bash
       git clone https://github.com/regro-cf-autotick-bot/livvkit-feedstock
       cd livvkit-feedstock
       git checkout 3.0.0
       ```
       make your changes, push them, and then rerender the feedstock. NOTE: make sure
       to add yourself as a maintainer to the recipe!  
     * once all tests pass, you can merge it into the feedstock. 
     
1. Request new LIVVkit version be included in the next `e3sm_unified` version here:
   https://acme-climate.atlassian.net/wiki/spaces/EIDMG/pages/129732419/Packages+in+the+E3SM+Unified+conda+environment

1. If/Once EVV works with this release, request new LIVVkit version be included in 
   the next `cime_env` version here: https://github.com/E3SM-Project/e3sm-unified/blob/master/recipes/cime-env/meta.yaml
   
# EVV 



# E3SM Core Efforts



