# expense_tracker
Decentralized Expense Tracker App(dapp)

A decentralized application (DApp) built with **Solidity**, **React**, and **Ethers.js**, deployed on the **Sepolia Testnet**. This app allows users to track expenses and interact with a smart contract to manage user data securely and transparently.
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
// --- LOAD ALL EXPENSES ---
  // Gets all expenses from the blockchain
  const loadExpenses = useCallback(async () => {
    if (!contract || !isRegistered) return; // Skip if not ready
    setLoadingExpenses(true); // Show loading indicator
    try {
      // Get the total number of expenses
      const count = await contract.expenseCount();
      const loaded = [];

      // Loop through each expense and load its details
      for (let i = 0; i < count; i++) {
        try {
          // Get basic expense info
          const [id, label, timestamp] = await contract.getExpenseBasicInfo(i);
          // Get list of addresses involved in this expense
          const participantsAddresses = await contract.getExpenseParticipants(
            i
          );

          // For each participant, get how much they paid and owe
          const participantsData = await Promise.all(
            participantsAddresses.map(async (address) => {
              try {
                const amountPaid = await contract.getAmountPaid(i, address);
                const amountOwed = await contract.getAmountOwed(i, address);
                return {
                  address,
                  amountPaid: ethers.utils.formatEther(amountPaid), // Convert from wei to ETH
                  amountOwed: ethers.utils.formatEther(amountOwed), // Convert from wei to ETH
                };
              } catch (error) {
                console.error(
                  `Error loading amounts for participant ${address}:`,
                  error
                );
                return { address, amountPaid: "0", amountOwed: "0" };
              }
            })
          );

          // Add this expense to our list
          loaded.push({
            id: id.toNumber(), // Convert from BigNumber to regular number
            label,
            timestamp: new Date(timestamp.toNumber() * 1000).toLocaleString(), // Convert timestamp to readable date
            participants: participantsData,
          });
        } catch (error) {
          console.error(`Error loading expense ${i}:`, error);
        }
      }

      setExpenses(loaded); // Save all expenses to state
    } catch (error) {
      console.error("Error loading expenses:", error);
      alert("Could not load expenses. Check console.");
    } finally {
      setLoadingExpenses(false); // Hide loading indicator
    }
  }, [contract, isRegistered]);

  // --- LOAD ALL REGISTERED PEOPLE ---
  // Gets list of all registered users
  const loadPeople = useCallback(async () => {
    if (!contract) return; // Skip if contract isn't ready
    try {
      // Get all registered addresses
      const addresses = await contract.getAllRegisteredPeople();
      // For each address, get their name and balance
      const peopleData = await Promise.all(
        addresses.map(async (address) => {
          const person = await contract.getPerson(address);
          const netBalance = await contract.getNetBalance(address);
          return {
            address,
            name: person.name,
            netBalance: ethers.utils.formatEther(netBalance), // Convert from wei to ETH
          };
        })
      );
      setPeople(peopleData); // Save people to state
    } catch (error) {
      console.error("Error loading people:", error);
    }
  }, [contract]);

  // ---TOTAL NO OF REGISTERED USERS ---
  //Showstotal no of registered users
  const loadTotalUsers = useCallback(async () => {
    if (!contract) return;
    try {
      const userCount = await contract.getTotalUsers();
      setTotalUsers(userCount.toNumber());
    } catch (error) {
      console.error("Error loading total users:", error);
    }
  }, [contract]);
  

  // --- RUNS WHEN CONTRACT OR ACCOUNT CHANGES ---
  // Checks if user is registered and loads their data
  useEffect(() => {
    const checkRegistration = async () => {
      if (!contract || !account) return; // Skip if contract or account isn't ready

      try {
        // Ask the contract if this user is registered
        const person = await contract.getPerson(account);
        const registered =
          person.walletAddress !== ethers.constants.AddressZero;
        setIsRegistered(registered);

        if (registered) {
          setName(person.name); // Save the user's name
          await loadExpenses(); // Load all expenses
          await loadPeople(); // Load all registered people
        }
        // Load total registered users after checking registration
        await loadTotalUsers();
      } catch (error) {
        console.error("Error checking registration:", error);
      }
    };
    checkRegistration();
  }, [contract, account,loadExpenses, loadPeople, loadTotalUsers]); // Run this when contract or account changes
