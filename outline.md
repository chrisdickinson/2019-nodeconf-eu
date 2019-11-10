# TODO

- watch economics of package management and TAKE NOTES

# what's wrong with javascript package management, or
# whew, it's really been A Year

- this talk is a sequel to ceej's "economics of open source"
    - hopefully in a james cameron-y way (entertaining: can follow along
      without the original material: aliens, terminator 2), but more likely in
      a tolkien-y way (the simarillion)
- let's set the stage. the story thus far:
    - in the last ten years, we've built the largest package ecosystem on earth
        - we've reinvented how we build apps, build websites, run open source projects, teach new programmers
        - we've done this mostly for free, mostly for each other
        - the dependency graph for most applications is wide and deep, and represents trust
        - this is an unprecedented amount of trust and it makes some folks (rightfully) nervous
            - trust means we can leverage the efforts of multiple people within a space, but it means any one of those people can harm us
            - however, we tend to focus our worry about trust on the actors within the system, not on those providing the system
        - every modern javascript application depends on npm (yes, even the ones that use yarn)
    - we built this together on top of a privately-controlled, centralized registry
    - centralized registries are easy to build, expensive to scale
        - a successful registry system must strike a balance between individuals giving up control and the benefits they confer:
            - individuals do not have final say over the name of their package
                - if I want to publish a new package called lodash, I can't. back to the drawing board
            - once published, code can't be pulled from the registry
                - if I want to pull my packages because I do not agree with the ethics of the registry, I can't
            - the registry itself can replace any code removed or metadata pointing at bad versions
                - if I wish to install a bitcoin miner as part of my popular package, well.
        - you give up certain capabilities to get certain benefits. overall, good thus far!
        - note that npm has to insert itself (through support, through tooling) into each of these balances, though
            - further note that the community doesn't get a say in setting these rules
            - bad tradeoffs are embedded in the good tradeoffs
        - in order to be the one with the final say in what "lodash" is, npm has to handle all publications of lodash,
          and it's got to be the authority for all mirrors of lodash (if folks are, indeed, even using a mirror.)
        - these servers cost money, this bandwidth costs money, maintaining this code costs money
    - npm has had, let's say, some trouble in the last year or so
    - to briefly recap:
        - the loss of most of its engineering & support departments
        - an NLRB settlement after union-busting
        - C-level and management changeover
        - the wombat was replaced by gak
            - THE WOMBAT. WAS REPLACED. BY GAK.
    - I stayed as long as I could, probably longer than I should've
    - I left npm around 4/19 to join eaze
        - ceej and i decided to do something about package management, because the problem is close to our hearts
        - we wrote the first version of entropic together
        - ceej announced it in may at jsconf eu on 6/5
            - I sweated bullets while making a live change to the deployment during the talk (ha ha ha)
- problem solved, right?
    - we had roughly 5 weeks to work on entropic: 3 off-time between jobs for me, 2 weeks after work while onboarding at eaze
    - decentralized registries are hard to build, cheap(er) to scale
        - as aforementioned, a good registry system is built from axiomatic tradeoffs
        - many of these tradeoffs -- you can't remove code that's been published, say -- are a lot harder when there's
          no central authority holding the keys
    - we took time to dissect these tradeoffs, and here's what we came to
        - (CD NOTE: this is where I get into the technical description of what The Thing is)
        - packages are email-like: `user@host/package`
        - packages replicate between instances on use (if you use something from another host, it is replicated)
        - package content is signed by multiple parties
            - by the original author namespace
            - by the receiving ENTROPIC
            - by downstream ENTROPICs
        - problem: entropic instances aren't forever, but package content must be
            - solution: whenever two entropics first meet, they trade public keys and consult with a ledger of host + pubkey
            - early on, the idea was to use keybase + a centralized, client-verifiable log
            - current thinking: we'll use a distributed ledger
        - package content is content-addressable (no more tarballs)
            - good on a lot of axes
            - this means there's something deterministic to sign
            - because our access patterns are mostly around "syncing" - from client to server or server to server - this
              makes the system gradually faster over time
            - but it also means you could build an outside service to say "these content hashes are harmful" -- security!
            - casualty: download counts
        - problem: we built a lot of value on top of the centralized registry (starting from scratch is a non-starter)
            - solution: a special "user" per-instance called `legacy` that proxies back
            - packages are copied and presented as `legacy@<url>/<package>`
            - for example: `legacy@registry.entropic.dev/lodash`, or `legacy@registry.entropic.dev/@hapi/joi`
    - we announced... and...
    - phew
        - we pretty quickly started drawing attention
        - work started ramping up just as we were also working full-time on entropic in our off hours
        - wham, we hit a wall
            - i started feeling the pinch when I pulled an all-nighter splitting the registry codebase into services ahead of
              a vacation (https://github.com/entropic-dev/entropic/issues/160)
            - taped over this with a policy of "no working on weekends"
        - we ground forward and tried to make ourselves work more in public channels (discourse, discord)
        - we were feeling the pointy end of the economics of open source
            - many decisions to be made + a lot of enthusiasm means a lot of attention was required, especially around moderation
              and education
            - bad feedback loop: dayjobs starved maintainers of available attention, moderation and education were the most urgent
              attention sinks ("keep the wheels on"), starving decision-making of available attention
                - spin up more maintainers?
                - you need the existing maintainers on the same page!
        - ultimately this ground the project to a halt
- code is never the challenge
    - deploy targets
        - moving out from a company means you leave your protective hermit crab shell
        - your deploy tooling usually doesn't come with you
        - as a result, the deploy target for entropic was very ad-hoc
        - people need concrete feedback loops
            - if you are building a service and it runs locally, that proves relatively nothing
            - there's a subtle, growing stress born from working on something that doesn't see deployment
              in its final form _somewhere_
            - this contributed to the attention-starving feedback loop. the website looked bad to get folks to work on the
              website, but there wasn't a clear way to work on the website, so folks spun their wheels on solutions
    - communication
        - from experience, ceej and I are a very effective team HOWEVER,
            - we are not the best at communicating intent out
            - communicating how the project made tactical decisions about, e.g., which packages to use occurred to us late
                - laissez-faire until we're not laissez-faire
            - keeping ourselves aligned is important, too
                - switching communications channels (to discord) severed that line for entropic
        - guilt about PRs lingering meant merging things before they were ready
            - moving the project forward this way made it move faster than it should
            - produced thrash
            - rfc for project cadence
            - introduced a merge queue and tied it to the project cadence lifecycle
                - better tests would help as well
                - but I note: "working more" cannot be the lesson we take
    - control
        - among my failings: I am at my happiest when I control the parameters in which code is written
            - see also: we built a framework (using the great packages from fastify and js-http!)
        - this has downsides: it's hard to communicate "decisions I want to be involved in" vs. "decisions I don't need to be part of"
            - this also creates a loop for me: if I see someone say something incorrect about the project I am compelled to (gently) correct it
                - but this also stresses me out because there's no finish line
                - and the longer I let something sit, the more I feel like something Bad Will Happen
        - ultimately: is this a project governed by a collaborator base or by benevolent dictators?
            - right now: it's benevolent dictators
                - this might disappoint some folks involved with the project
                - however. we don't want to be benevolent dictators for life, just until the project works
                - but maybe we tried to do this too soon
    - too much talking about the project, too much stress about the project, not enough work on the project
        - projects -- and companies, really -- are like bicycles: without forward progress, they start to wobble
            - eventually they'll fall over
- so what have we learned
    - this is (still) important
        - we traded control of our commons to a private entity
        - in the best case, Microsoft (or someone else with deep pockets) takes over the centralized registry
            - They're friendly now, but is any single private organization trustworthy with that kind of power over our commons?
            - how comfortable are you with one company controlling the horizontal and vertical on development?
    - swallow our guilt about failing the project collaborators
        - I am here now, eating the proverbial crow and admitting my failings to you because I think this is
          still worth our communities' attention.
    - pick the bike up and start peddling
        - weekly meetings between ceej and I (for now)
        - point the existing codebase at a (repeatable) deploy target ASAP. Probably kubernetes. End goal of this project is to become an appliance
          that you can just Plug In and Run.
        - get the project _into_ a state where collaborators can come on board & make forward progress without stressing out maintainers
- key takeaway: we're building a protocol, not a program
    - most important way to get involved is:
        - help spec the protocol
        - build ref implementations of the protocol
        - we pick goals: it should cost $5 or less to run entropic for a personal use
            - $10-50 for larger groups and companies



