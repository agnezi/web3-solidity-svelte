<script>
	// @ts-nocheck

	import { ethers } from 'ethers';
	import { onMount } from 'svelte';
	import TipJarABI from '../../artifacts/src/contracts/TipJar.sol/TipJar.json';

	let userAddress = null;
	let network = null;
	let balance = null;
	let isConnected = false;
	let provider = null;

	const contractAddress = import.meta.env.VITE_CONTRACT_ADDRESS;
	let contract = null;
	let allTips = [];
	let isWithdrawing = false;

	let amItheOwner = false;

	let contractBalance = null;

	async function setupContract() {
		if (isConnected) {
			contract = new ethers.Contract(contractAddress, TipJarABI.abi, provider);
			const contractOwner = await contract.owner();
			console.log(contractOwner);
			amItheOwner = ethers.utils.getAddress(contractOwner) === ethers.utils.getAddress(userAddress);
			console.log(amItheOwner);
			contract.on('NewWithdraw', async () => {
				contractBalance = await provider.getBalance(contractAddress);
				balance = await provider.getBalance(userAddress);
				isWithdrawing = false;
			});
		}
	}

	async function getAllTips() {
		if (isConnected) {
			const tips = await contract.getAllTips();
			allTips = tips.map((item) => {
				return {
					...item,
					timestamp: new Date(item.timestamp * 1000).toLocaleDateString(),
					amount: ethers.utils.formatEther(item.amount.toString())
				};
			});
		}
	}

	async function setup(accounts) {
		userAddress = accounts[0];
		try {
			provider = new ethers.providers.Web3Provider(window.ethereum);
			network = await provider.getNetwork();
			balance = await provider.getBalance(userAddress);
			contractBalance = await provider.getBalance(contractAddress);
			isConnected = true;
			setupContract();
		} catch (error) {
			console.error(error);
		}
	}

	async function connectWallet() {
		if (window.ethereum) {
			const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });

			if (accounts.length > 0) {
				await setup(accounts);
			} else {
				alert('No accounts found');
			}
		} else {
			alert('No ethereum wallet found');
		}
	}

	onMount(async () => {
		if (window.ethereum) {
			const accounts = await window.ethereum.request({ method: 'eth_accounts' });
			if (accounts.length > 0) {
				await setup(accounts);
			}
		}
	});

	async function withdraw() {
		if (isConnected) {
			isWithdrawing = true;

			const rwContract = new ethers.Contract(contractAddress, TipJarABI.abi, provider.getSigner());
			const tx = await rwContract.withdraw();
			await tx.wait();
		}
	}
</script>

{#if isConnected}
	<h1 class="text-3x1 text-gray-800 p-8">Withdraw from the TipJar contract</h1>
	<div class="text-sm text-gray-500 pb-4 flex flex-col gap-4">
		<p class="text-xl text-green-600">
			Successfully connected with account: <strong>{userAddress}</strong>
		</p>
		<ul>
			<li>Current network: {network.name}</li>
			<li>Current balance: {ethers.utils.formatEther(balance)} eth</li>
			<li>Current contract balance: {ethers.utils.formatEther(contractBalance)} eth</li>
		</ul>

		{#if amItheOwner}
			{#if contractBalance.eq(0)}
				<p class="text-xl text-red-600">There is no balance to withdraw</p>
			{/if}
			<button
				class="bg-blue-600 text-gray-50 shadow-md rounded-md px-3 py-8 text-center disabled:opacity-25"
				on:click={withdraw}
				disabled={isWithdrawing || contractBalance.eq(0)}
			>
				{isWithdrawing ? '...Withdrawing' : 'Withdraw'}
			</button>
		{:else}
			<p class="text-xl text-red-600">Only the owner of the contract can withdraw the balance</p>
		{/if}
	</div>
{/if}
