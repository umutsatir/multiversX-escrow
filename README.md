# MultiversX Escrow Contract

A secure and efficient escrow smart contract built on the MultiversX blockchain, enabling safe peer-to-peer transactions with escrow functionality.

## Overview

This escrow contract provides a trustless way to conduct transactions between two parties on the MultiversX blockchain. It allows a seller to create an offer by locking EGLD tokens in the contract, which can then be accepted by a specific buyer. The contract ensures that funds are only released when the buyer accepts the offer, or returned to the seller if they cancel it.

## Features

- **Create Escrow Offers**: Lock EGLD tokens in the contract with a designated recipient.
- **Accept Offers**: Recipients can accept offers to receive the locked tokens.
- **Cancel Offers**: Creators can cancel their active offers and retrieve their tokens.
- **View Functions**: Query active offers, user offers, and incoming offers.
- **Event Logging**: All actions (create, accept, cancel) are logged on the blockchain for transparency.

## Smart Contract Structure

### Data Types

- **OfferStatus**: Enum defining the possible states of an offer (Active, Completed, Cancelled).
- **Offer**: Struct containing offer details including creator, recipient, amount, status, etc.

### Public Endpoints

1. **create**: Creates a new escrow offer by locking EGLD tokens.
   ```
   #[payable("EGLD")]
   #[endpoint]
   fn create(&self, buyer: ManagedAddress)
   ```

2. **acceptOffer**: Allows the recipient to accept an offer and receive the locked tokens.
   ```
   #[endpoint(acceptOffer)]
   fn accept_offer(&self, offer_id: u64) -> SCResult<()>
   ```

3. **cancelOffer**: Allows the creator to cancel an active offer and receive a refund.
   ```
   #[endpoint(cancelOffer)]
   fn cancel_offer(&self, offer_id: u64) -> SCResult<()>
   ```

### View Functions

1. **getLastOfferId**: Returns the ID of the last created offer.
   ```
   #[view(getLastOfferId)]
   fn last_offer_id(&self) -> SingleValueMapper<u64>;
   ```

2. **getOffer**: Returns detailed information about a specific offer.
   ```
   #[view(getOffer)]
   fn offer(&self, id: u64) -> SingleValueMapper<Offer<Self::Api>>;
   ```

3. **getActiveOffers**: Returns all active offers in the contract.
   ```
   #[view(getActiveOffers)]
   fn get_active_offers(&self) -> MultiValueEncoded<Offer<Self::Api>>
   ```

4. **getUserActiveOffers**: Returns all active offers created by a specific user.
   ```
   #[view(getUserActiveOffers)]
   fn get_user_active_offers(&self, user: &ManagedAddress) -> MultiValueEncoded<Offer<Self::Api>>
   ```

5. **getUserIncomingActiveOffers**: Returns all active offers addressed to a specific user.
   ```
   #[view(getUserIncomingActiveOffers)]
   fn get_user_incoming_active_offers(&self, user: &ManagedAddress) -> MultiValueEncoded<Offer<Self::Api>>
   ```

### Events

The contract emits the following events:
- `createOffer`: When a new offer is created
- `cancelOffer`: When an offer is cancelled
- `acceptOffer`: When an offer is accepted

## How It Works

1. **Creating an Offer**:
   - A seller calls the `create` endpoint with a buyer's address and sends EGLD.
   - The contract generates a unique offer ID and stores the offer details.
   - The offer is added to both the seller's and buyer's offer lists.

2. **Accepting an Offer**:
   - The designated buyer calls the `acceptOffer` endpoint with the offer ID.
   - After validation, the contract transfers the locked EGLD to the buyer.
   - The offer status is updated to "Completed".

3. **Cancelling an Offer**:
   - The seller calls the `cancelOffer` endpoint with the offer ID.
   - After validation, the contract returns the locked EGLD to the seller.
   - The offer status is updated to "Cancelled".

## Security Features

- **Address Verification**: Only the correct buyer can accept an offer and only the creator can cancel it.
- **Status Validation**: Offers can only be accepted or cancelled if they are in the "Active" state.
- **Safe Token Transfers**: The contract uses MultiversX's secure transfer functions.
- **Thread-safe Operations**: Uses safe storage methods like `set_if_empty()`.
