# Web3 and Svelt study

## Follow the steps to run this project:

- First run `yarn hardhat:compile`
- Second in another terminal run `yarn hardhat node` and let it running
- Third run `yarn hardhat:deploy` and copy the *Tip Jar deployed, address*
  - *IMPORTANT* Create an .env file on the root of this project, add a variable with name `VITE_CONTRACT_ADDRESS` and paste address previous copied to this env variable
- Fourth run `yarn dev`