# Joe's ORNL Projects
*and how to maintain them with his ~~demise~~ departure*


Joe's projects/tasks at ORNL can be grouped into three main categories, which 
are briefely described here. 

Use [this repositories issues](https://github.com/jhkennedy/JORNL/issues) 
to ask questions, get clarification, etc. about any 
of these. 

## LIVVkit

LIVVkit, as a project, has two basic parts. verification of Ice Sheet Models 
(ISMs) and validation of ISMs. Right now, LIVVkit is funded through the ProbSLR
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


## EVV 



## E3SM Core Efforts



