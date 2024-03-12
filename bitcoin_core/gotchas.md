# "Gotchas" when Onboarding
Below is a running list of "gotchas" that one may want to look out for when onboarding into Bitcoin Core.  As more info is gleamed over time, references will be added.

## Historical Context
Bitcoin development is distributed, not centralized, so there is no strict project management structure, and no formal training process.  Sometimes it's difficult to have all the historical context.  For example, in [PR 29530](https://github.com/bitcoin/bitcoin/pull/29530#issuecomment-1978887560) sipa had the historical context of an intent to remove the concept of a misbehavior/ban score and the last steps in doing this weren't finished.

https://bitcoinops.org/ provides some recaps and history.  Not sure if there are other resources available that would help onboarders avoid retreading past ground.

Perhaps increased documentation in the `doc` portion of the repo, specific to the functional area of the design, could help.  For example, a design document for P2P might include the current design approach as well as a "historical decisions" or "deprecated approaches" section.  A risk being that documentation and actual code get out of sync.

## Rebasing
Don't perform extraneous rebases.  A rebase invalidates existing ACKs, so once these exist it should only be done if there is a conflict (Drahtbot will complain) or a specific reason (such as a silent conflict). The CI will always rebase on master anyway for its runs (which could be restarted without rebasing if really necessary).
