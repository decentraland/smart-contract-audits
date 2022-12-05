# Rental Contracts

Date: 2022-11-10 <br>
Author: Facundo Spagnuolo<facundo.spagnuolo@gmail.com> <br>
Commit: [`ad906381b96f23c0b82a9ba89cf3466914448139`](https://github.com/decentraland/vestings-builder/tree/ad906381b96f23c0b82a9ba89cf3466914448139) <br>

## 0. Assumptions

#### 0.1. Configuration

The contract does not perform any specific on-chain validation regarding its configuration, it only checks these values
are set which is enough to ensure the contract correct behavior. However, it is assumed the contract creator will use
reasonable number of vesting periods, a reasonable cliff duration, and a reasonable period duration.

## 1. Critical

None

## 2. High

None

## 3. Medium

#### 3.1. Revoking extends the vesting end date when paused

`PeriodicTokenVesting` defines two functions `revoke` and `pause` that can limit the vesting contract end date, with 
the difference that the former is irreversible. If for some reason the `owner` decides to pause the contract, and 
finally revoke it in a future date, the vested amount of tokens will be computed considering the date when the contract
was revoked instead of using the date when the contract was paused.

Consider not overriding the `stop` timestamp if it was set when revoking, or at least forcing the `owner` to unpause 
the contract before revoking it which can be done by adding the `whenNotPaused` modifier to the `revoke` function.

**Update:** Fixed at [`c8049ad98b543196647b2418dcca16cd3624009a`](https://github.com/decentraland/vestings-builder/commit/c8049ad98b543196647b2418dcca16cd3624009a)

## 4. Low

#### 4.1. Start date can be set to zero

`PeriodicTokenVesting` allows initializing the contract with a `start` variable set to zero. Even though this does not
impact in any of the calculations performed in the contract, it can be an undesired scenario if the creator forgets to
enter a reasonable start date.

Similarly to how other variables are checked during the contract's initialization, consider making sure `start` is not 
zero.

**Update:** Fixed at [`b3fa99c1758e962f77f3ec8ef64ba5d223765991`](https://github.com/decentraland/vestings-builder/commit/b3fa99c1758e962f77f3ec8ef64ba5d223765991)

## 5. Notes

#### 5.1. Cannot run tests

Running `npm test` or `yarn test` does not do anything. Trying to run tests with `yarn hardhat test` also fails due to
some dependencies issues.

Consider either providing a default test script that works or mentioning in the README file how tests are supposed to
be run.

**Update:** Acknowledge  

#### 5.2. Misleading naming convention for some variables

There are a few scenarios where past tenses are used to name specific variables. This is the case of `vestedThisPeriod`
or `vestedPerPeriod` for example. Using a past tense here reflects as if the variable is treating a number of tokens 
that where already vested, when it actually has to be with a configuration.  

`vestedThisPeriod` is used in `L277` and it actually refers to the number of tokens to vest in the current period which
is meaningfully different.

The same occurs with the storage variable defined in `L42` named `vestedPerPeriod` which actually refers to the number 
of tokens to be vested per period, and not the number of tokens that were vested per period.

Consider renaming these variables to avoid misleading the reader.

**Update:** Acknowledge