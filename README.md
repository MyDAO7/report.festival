# report.festival
FestivalPass: ERC1155 Smart Contracts for Ticketing, Rewards &amp; NFT Memorabilia
Prepared by: [Malaika Sherazi]
Date: [20/07/25]

Executive Summary:
We’ve audited the FestivalPass ERC1155 contract, which handles festival passes, performances, and NFT memorabilia. The code is generally well-structured, but we found a few critical security gaps—especially around access control and input validation—that need immediate attention.

Key Risks:

1-: Unchecked performance IDs could lead to fake rewards.

2-: The withdraw() function is a free-for-all; funds could be stolen if the owner is compromised.

3-: Unbounded loops might brick the getUserMemorabiliaDetailed() function.



Detailed Findings

**1- Attending Non-Existent Performances (WTF?)**
Where: attendPerformance()

What’s Broken: You can "attend" a performance that doesn’t exist. The contract doesn’t check if performanceId is valid before minting rewards.

Worst Case: Scammers could drain BeatToken by spamming fake performance IDs.

Fix:

solidity
require(performances[performanceId].startTime != 0, "Performance doesn’t exist");
**2- Unsafe ETH Withdrawal (Seriously?)**
Where: withdraw()

What’s Broken: The owner can send all ETH to any address, no questions asked. If the owner’s key leaks, say goodbye to funds.

Worst Case: A hacked owner = all ETH gone.

Fix:

Use a timelock or multisig for withdrawals.

At minimum, add a reentrancy guard:

solidity
bool private _locked;
modifier noReentrancy() {
    require(!_locked, "No reentrancy");
    _locked = true;
    _;
    _locked = false;
}
**3- Memorabilia Collection Overflow (Oops.)**
Where: redeemMemorabilia()

What’s Broken: The supply check (currentItemId < maxSupply) could fail silently if currentItemId overflows.

Worst Case: Someone mints unlimited NFTs, crashing the collection.

Fix:

solidity
require(collection.currentItemId <= collection.maxSupply, "Sold out");
**4- Anyone Can Be the Organizer (Wait, What?)**
Where: setOrganizer()

What’s Broken: The function is public but should be onlyOwner. Right now, any user can appoint themselves as the organizer.

Worst Case: A rogue organizer messes up performances, rewards, or NFT collections.

Fix:

solidity
function setOrganizer(address _organizer) external onlyOwner { ... }
3. Medium-Risk Issues
3.1 Gas Bomb in getUserMemorabiliaDetailed()
Where: getUserMemorabiliaDetailed()

What’s Broken: Loops through all possible NFTs. If a user holds 1,000 NFTs, this function will fail.

Fix:

Paginate results or offload this to an indexer (e.g., The Graph).


**5- Gas Wasters**
Cache collections[collectionId] (saves ~2K gas per read).

Use unchecked for safe math (e.g., ++passSupply can’t overflow due to maxSupply checks).

The contract’s design is solid, but the devil’s in the details. A few small oversights could lead to big exploits. Prioritize the critical fixes, then clean up the rest.
