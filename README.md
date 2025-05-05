# BFL-Cartesi: Blockchain Federated Learning with Challenge Mechanism

## Overview

BFL-Cartesi is a proof-of-concept implementation of a blockchain-based federated learning system with a challenge mechanism, built on the Cartesi optimistic rollup platform. This project demonstrates how multiple aggregators can participate in a federated learning process with validators that can challenge potentially incorrect aggregations.

## Key Concepts

- **Blockchain Federated Learning**: Combines federated learning with blockchain for transparent, decentralized model training
- **Optimistic Aggregation**: Aggregations are accepted by default but can be challenged within a time window
- **Challenge Mechanism**: Validators can challenge aggregator results, with penalties for incorrect aggregation
- **Deterministic Verification**: Cartesi enables deterministic computation verification on-chain
- **Round-Robin Aggregation**: Aggregators take turns for model aggregation in a predefined order
- **Verifiable Transparency**: Cartesi notices provide on-chain evidence of computation outcomes

## Tools and Technologies

- **Flower**: A federated learning framework for the core FL functionality
- **PyTorch**: Deep learning framework for model training and evaluation
- **Cartesi**: Optimistic rollup platform for verifiable computation
- **Python**: Primary programming language for the implementation

## Architecture

The system follows this workflow:

1. Clients train local models and submit updates to the blockchain
2. Aggregators (in round-robin order) perform FedAvg aggregation and submit results
3. Validators can challenge aggregation results within a time window
4. The challenge is verified by re-computing the aggregation
5. If the challenge is successful, the aggregator is banned and the model is rolled back

## Core Components

### Cartesi DApp (`dapp.py`)

The Cartesi DApp serves as the on-chain component that handles federated learning operations and the challenge mechanism.

#### Key Functions:

- `handle_submit_update(data)`: Process client model updates
- `handle_submit_aggregation(data)`: Process aggregator results
- `handle_submit_challenge(data)`: Process validator challenges
- `verify_challenge(challenge_id)`: Verify challenge by re-computing aggregation
- `fedavg_aggregate(updates)`: Implement FedAvg algorithm for verification
- `handle_next_round(data)`: Advance to the next FL round
- `handle_initialize(data)`: Initialize the system with aggregators

### Enhanced Validation with Cartesi Notices

The validation mechanism has been enhanced using Cartesi's notice functionality to improve transparency, auditability, and trust:

#### Transparent Challenge Processing

- **Challenge Submission Notices**: When a validator submits a challenge, a notice is emitted to the blockchain containing the challenge details, timestamp, validator ID, and aggregator ID.
- **Challenge Verification Notices**: During verification, detailed metrics and comparison results are published as notices without exposing full model parameters.
- **Challenge Resolution Notices**: The outcome of each challenge is recorded on-chain through notices, providing immutable evidence of the verification process.

#### Validation Metrics

The system uses several key metrics published as notices for validation:

- **Euclidean Distance**: Measures overall parameter deviation between claimed and correctly computed aggregations
- **Cosine Similarity**: Evaluates the directional alignment of parameters
- **Parameter Discrepancy Analysis**: Identifies and reports the most significant differences in model parameters
- **Statistical Validation**: Applies statistical tests to determine if differences are within acceptable thresholds

#### Verification Mechanism

Verification follows these steps, with notices published at each stage:

1. **Pre-Verification Notice**: Announces the beginning of verification with challenge details
2. **Computation Notice**: Records the hash of re-computed aggregation for later verification
3. **Comparison Notice**: Details similarity metrics between claimed and correct aggregations
4. **Evidence Notice**: Publishes cryptographic proofs of the verification process
5. **Resolution Notice**: Records the final decision with supporting evidence

#### Notice Schema

All notices follow a structured schema for easy consumption by blockchain clients:
```json
{
  "type": "[notice_type]",
  "round_id": "[round_identifier]",
  "timestamp": "[UTC_timestamp]",
  "data": {
    // Notice-specific data
  },
  "hash": "[cryptographic_hash_of_evidence]"
}
```

### Client Simulator

Simulates FL clients for testing purposes.

#### Key Functions:

- `train_local_model()`: Train model on local data
- `submit_update(round_id)`: Submit model update to Cartesi DApp
- `get_global_model()`: Retrieve current global model
- `evaluate_global_model()`: Evaluate global model on local test data

### Aggregator Simulator

Simulates aggregators for testing purposes.

#### Key Functions:

- `submit_aggregation(round_id, weights)`: Submit aggregation result

### Malicious Aggregator Simulator

Extends the Aggregator Simulator to test the challenge mechanism.

#### Key Functions:

- `submit_malicious_aggregation(round_id, weights)`: Submit intentionally incorrect aggregation

### Validator Simulator

Simulates validators for testing the challenge mechanism.

#### Key Functions:

- `submit_challenge(round_id)`: Challenge an aggregation result
- `get_state()`: Retrieve current system state
- `verify_notices()`: Verify the correctness of emitted notices

### Simulation Script

Orchestrates the entire FL process with challenges for testing.

#### Key Functions:

- `run_honest_simulation()`: Run simulation with honest aggregators
- `run_malicious_simulation()`: Run simulation with a malicious aggregator

## Implementation Plan

### Phase 1: Core Functionality

1. **Set up Cartesi DApp**
   - Implement basic request handling
   - Define data structures for FL state

2. **Implement FL Operations**
   - Client update submission
   - Aggregator result submission
   - Round management

### Phase 2: Challenge Mechanism

1. **Implement Challenge Logic**
   - Challenge submission with notice emission
   - Verification computation with detailed metrics
   - Result determination with cryptographic evidence

2. **Implement Penalties**
   - Ban system for malicious participants
   - Rollback mechanism for invalid aggregations
   - Transparent penalty enforcement through notices

### Phase 3: Testing

1. **Create Simulation Environment**
   - Honest simulation
   - Malicious aggregator simulation
   - Verification of challenge outcomes
   - Notice verification and analysis

## Test-Driven Development

The implementation follows TDD principles with these key test scenarios:

1. **Honest Aggregation Test**
   - All aggregators behave honestly
   - No challenges are issued
   - Global model converges normally
   - Notices confirm proper state transitions

2. **Malicious Aggregation Test**
   - One aggregator submits incorrect results
   - Validator challenges the result
   - System verifies the challenge and bans the malicious aggregator
   - Notices provide transparent evidence of verification

3. **System State Test**
   - Verify correct state transitions
   - Confirm banned participants list integrity
   - Validate round progression
   - Analyze notice trail for completeness

4. **Evidence Verification Test**
   - Verify cryptographic proofs in notices
   - Confirm calculation consistency
   - Validate decision thresholds

## Running the PoC

### Prerequisites

- Python 3.8+
- Docker
- Cartesi node

### Setup

1. Clone the repository:
```bash
git clone https://github.com/yourusername/BFL-cartesi.git
cd BFL-cartesi
```

2. Build the Docker image:
```bash
docker build -t bfl-cartesi .
```

3. Run the Cartesi node:
```bash
cartesi run bfl-cartesi
```

### Testing

1. Connect to the running container:
```bash
docker exec -it bfl-cartesi bash
```

2. Run the simulation:
```bash
python run_simulation.py
```

## Expected Outputs

### Honest Simulation

- All rounds complete successfully
- No challenges are raised
- No participants are banned
- Notices confirm proper execution

### Malicious Simulation

- Malicious aggregator submits incorrect aggregation
- Validator challenges the result
- Challenge verification succeeds with detailed metrics in notices
- Malicious aggregator is banned
- System continues with remaining honest aggregators
- Complete audit trail available through notices

## Monitoring and Transparency

### Notice Explorer

The system includes a simple web interface to explore notices:

1. **Challenge Timeline**: Visualize the sequence of challenges and resolutions
2. **Metrics Dashboard**: View validation metrics for each challenge
3. **Evidence Verification**: Verify cryptographic proofs of computations
4. **Participant Status**: Monitor participant standing and reputation

### Integration with Blockchain Explorers

- Notices are indexed for easy querying through standard blockchain explorers
- Each notice includes references to related notices for audit trail following
- Cryptographic proofs enable independent verification of all decisions

## Future Enhancements

1. **Economic Incentives**: Implement token rewards/penalties for participants
2. **Multiple Validators Consensus**: Require multiple validators to confirm a challenge
3. **Advanced Aggregation Strategies**: Support beyond FedAvg
4. **Privacy-Preserving Techniques**: Add secure aggregation or differential privacy
5. **Dynamic Participant Management**: Allow new participants to join/leave
6. **Enhanced Notice Analytics**: Develop advanced tools for notice analysis and visualization

## Conclusion

This PoC demonstrates a viable approach to implement a challenge mechanism in blockchain federated learning systems using Cartesi's optimistic rollup platform. The enhanced validation system using Cartesi notices provides transparent, verifiable computation evidence while maintaining the efficiency of the optimistic execution model. This architecture enables a trust-minimized federated learning system with robust protection against malicious aggregators and transparent verification for all participants.