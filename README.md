# expense_tracker
Decentralized Expense Tracker App(dapp)

This is a decentralized application (DApp) built with Solidity, React, and Ethers.js and deployed on the Sepolia Testnet. It allows users to track expenses and interact with a smart contract to manage user data securely and transparently.
New features added in the app include:
*Solidity feature-
Get total number of registered people
Returns the total number of registered users on the platform.
code fix made:
function getTotalUsers() public view returns (uint256 count) {
    return registeredPeople.length;
    }

*Java Script feature-
Display total registered users
Retrieves and displays how many users are currently registered.
code fix made:
// ---TOTAL NO OF REGISTERED USERS ---
  //Shows total no of registered users
  const loadTotalUsers = useCallback(async () => {
    if (!contract) return;
    try {
      const userCount = await contract.getTotalUsers();
      setTotalUsers(userCount.toNumber());
    } catch (error) {
      console.error("Error loading total users:", error);
    }
  }, [contract]);
