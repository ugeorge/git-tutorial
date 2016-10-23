Step-by-step instructions
=========================

This tutorial assumes a Ubuntu Linux machine (or some Debian distribution), and uses a simple Haskell program as toy example. Prerequisites: `git`, `ghc`, `cabal-install`.

Setting up Git
--------------

1. configure local machine

        sudo apt-get install git
        git config --global user.name "name"
        git config --global user.email "name@kth.se"
    
        # check if existing generated keys
        ls -al ~/.ssh
    
        # generate ssh key if none present 
        ssh-keygen -t rsa -b 4096 -C "name@kth.se"
    
        # start the ssh-agent in the background and add the key
        eval "$(ssh-agent -s)"
        ssh-add ~/.ssh/id_rsa
    
        # copy the generated key found below to your GitHub account
        emacs ~/.ssh/id_rsa.pub


Basic Usage
-----------

1. create `demo` private repo on `gitr`. copy link

1. init a new local repo an link it to `demo`

        mkdir demo
        cd demo
        git init
        ls -la
        git remote add origin git@gitr.sys.kth.se:[name]/demo.git

1. create a software project

        cabal init
        emacs demo.cabal
        # uncomment and add exposed-modules: Toy

1. fill the project with some code. E.g. a Toy.hs Haskell file:

    module Toy where

        myrev []     = []
        myrev (x:xs) = myrev xs ++ [x]

1. stage the files for commit:

        git add .
        git status

1. whoops! unstage!
    
        git reset
        git status

1. add a `.gitignore` file and fill it with:

        *~
        *#
        .cabal*
        cabal*
        dist*

1. (optional) stash something...

1. stage an commit:

        git status
        git add .
        git status
        git commit -m "first commit"

1. push your commit to the `gitr` server
    
        git push

1. check the Web UI

Branching and merging
---------------------

1. New branches from the Web UI also but for this tutorial from CLI...

1. Create a branch `dev` locally:

        git branch dev
        git branch
        git checkout dev
        git branch

1. add a new function to `Toy.hs`:

        quicksort []     = []
        quicksort (x:xs) = [q | q <- xs, q < x] 
                           ++ [x] 
                           ++ [q | q <- xs, q > x]

1. stage, commit, push

        git status
        git add Toy.hs
        git commit -m "added function quicksort"
        git push 

        # mnope...
        git push origin dev

1. make some modifications in the `master` branch

        git checkout master
        emacs Readme.md
        #...
        git add Readme.md
        git commit -m "added readme file"
        git push origin master

1. see how everything looks on the Web UI

1. merge `dev` into `master`. Refer to [this page](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging) for more details. For now it suffices:

        git diff dev
        git merge dev

1. branch `dev` can be removed


Collaborative development
-------------------------

1. check the Web UI. Add a collaborator.

1. create an issue (#1)

> 1. collaborator clones the repo and creates an own branch

        cd [work-folder]
        git clone git@gitr.sys.kth.se:[name]/demo.git
        cd demo
        git checkout -b dev-collab

> 1. collaborator fixes the issue, and stages, commits and pushes the changes into the own branch

        git add .
        git commit -m "closes #1. Fixed the issue."
        git push origin dev-collab

> 1. collaborator issues this time a pull request

1. the merge is done online, through the Web UI

1. check the issue

1. now we want to go public. Make a public repo on GitHub. Add local remote to the public server.

        git remote add public <public-url>
        git push public master

1. external "benefactor" forks the public repository, commits changes and issues a pull request.

1. we honor the pull request and merge the changes online

1. it seemed that the "benefactor" messed things up! we need to revert to a previous commit. Refer to [this page](http://stackoverflow.com/questions/4114095/how-to-revert-git-repository-to-a-previous-commit) for more details. For now it is enough:

        git reset --hard [previous-SHA] 

1. tag the current version. Command line also available, but this time let's use the Web UI.


Continuous integration
----------------------

1. add a test suite.

1. add the following dependencies in `demo.cabal` under `build-depends:`

        test-framework,
        test-framework-hunit,
        test-framework-quickcheck2,
        QuickCheck, 
        HUnit

1. make Cabal know you are implementing a test suite. Add this at the end of the file:

        Test-Suite demo-testsuite
          Type:                exitcode-stdio-1.0
          Default-Language:    Haskell2010
          Hs-Source-Dirs:      .
          Main-is:             Test.hs
          Build-depends:       base >=4.9 && <4.10, 
                               demo, 
                               QuickCheck, 
                               test-framework,
                               test-framework-hunit,
                               test-framework-quickcheck2,
                               HUnit


1. Create `Test.hs`:

        module Main where

        import Test.Framework
        import Test.Framework.Providers.HUnit
        import Test.Framework.Providers.QuickCheck2 (testProperty)
        import Test.HUnit
        import Test.QuickCheck

        import Toy

        prop_myrev xs = (myrev . myrev) xs == xs
          where types = xs :: [Int]

        prop_quicksort xs = length (quicksort xs) == length xs
          where types = xs :: [Int]

        test_quicksort = quicksort "Ingo Likes Haskell" @?= "  HILaeegikkllnoss"

        tests = [testGroup "Demo Tests" [
                    testProperty "myrev . myrev = id" prop_myrev,
                    testCase "test quicksort" test_quicksort
                    ]
                ]
                     
        main = defaultMain tests


1. test locally:

        cabal test

1. introduce TravisCI

1. integrate the public `demo` with TravisCI. Copy/paste the `.travis.yml` file from [forsyde-atom](https://github.com/forsyde/forsyde-atom).

1. retry the scenario with the "benefactor"

Git goodies
-----------

1. if time permits it, try out some goodies
    
   * wiki pages
   * Graphs
   * GitHub pages


