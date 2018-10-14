# EurocoinCash
Código de repositorio para Eurocoin Cash, una criptomoneda con implementación de protocolo CryptoNote.

This is the Fork for the Cryptocurrency Eurocoin Cash - forking reference code from CryptoNote cryptocurrency protocol.

Plataform: several Ubuntu-based dedicated servers (at least 2Gb of RAM) for seed nodes.

Written in C++.

@facundomartin @sebanielsen

1. in file src/CryptoNoteConfig.h - CRYPTONOTE_NAME constant

const char CRYPTONOTE_NAME[] = "EurocoinCash";
2. in src/CMakeList.txt file - set_property(TARGET daemon PROPERTY OUTPUT_NAME "EurocoinCashd")

set_property(TARGET daemon PROPERTY OUTPUT_NAME "EurocoinCashd")
Emission logic

1. Total money supply

Total amount of coins to be emitted. Most of CryptoNote based coins use (uint64_t)(-1) (equals to 18446744073709551616). You can define number explicitly (for example UINT64_C(858986905600000000)).

Example:

const uint64_t MONEY_SUPPLY = (uint64_t)(-1);
2. Emission curve

Be default CryptoNote provides emission formula with slight decrease of block reward with each block. This is different from Bitcoin where block reward halves every 4 years.

EMISSION_SPEED_FACTOR constant defines emission curve slope. This parameter is required to calulate block reward.

Example:

const unsigned EMISSION_SPEED_FACTOR = 18;
3. Difficulty target

Difficulty target is an ideal time period between blocks. In case an average time between blocks becomes less than difficulty target, the difficulty increases. Difficulty target is measured in seconds.

Difficulty target directly influences several aspects of coin's behavior:

transaction confirmation speed: the longer the time between the blocks is, the slower transaction confirmation is
emission speed: the longer the time between the blocks is the slower the emission process is
orphan rate: chains with very fast blocks have greater orphan rate
For most coins difficulty target is 60 or 120 seconds.

const uint64_t DIFFICULTY_TARGET = 60;
4. Block reward formula

The implementation is in src/CryptoNoteCore/Currency.cpp:

bool Currency::getBlockReward(size_t medianSize, size_t currentBlockSize, uint64_t alreadyGeneratedCoins, uint64_t fee, uint64_t& reward, int64_t& emissionChange) const
This function has two parts:

basic block reward calculation: uint64_t baseReward = (m_moneySupply - alreadyGeneratedCoins) >> m_emissionSpeedFactor;
big block penalty calculation: this is the way CryptoNote protects the block chain from transaction flooding attacks and preserves opportunities for organic network growth at the same time.
Only the first part of this function is directly related to the emission logic.

Third step. Networking
1. Default ports for P2P and RPC networking

P2P port is used by daemons to talk to each other through P2P protocol. RPC port is used by wallet and other programs to talk to daemon.

Example:

const int P2P_DEFAULT_PORT = 17236;
const int RPC_DEFAULT_PORT = 18236;
2. Network identifier

This identifier is used in network packages in order not to mix two different cryptocoin networks. Change all the bytes to random values for your network:

const static boost::uuids::uuid CRYPTONOTE_NETWORK = { { 0xA1, 0x1A, 0xA1, 0x1A, 0xA1, 0x0A, 0xA1, 0x0A, 0xA0, 0x1A, 0xA0, 0x1A, 0xA0, 0x1A, 0xA1, 0x1A } };
3. Seed nodes

Add IP addresses of your seed nodes.

const std::initializer_list<const char*> SEED_NODES = {
  "111.11.11.11:17236",
  "222.22.22.22:17236",
};
Fourth step. Transaction fee and related parameters
1. Minimum transaction fee

Zero minimum fee can lead to transaction flooding. Transactions cheaper than the minimum transaction fee wouldn't be accepted by daemons. 100000 value for MINIMUM_FEE is usually enough.

const uint64_t MINIMUM_FEE = 100000;
2. Penalty free block size

CryptoNote Protocol use for EurocoinCash protects chain from tx flooding by reducing block reward for blocks larger than the median block size. However, this rule applies for blocks larger than CRYPTONOTE_BLOCK_GRANTED_FULL_REWARD_ZONE bytes.

const size_t CRYPTONOTE_BLOCK_GRANTED_FULL_REWARD_ZONE = 20000;
Fifth step. Address prefix
You may choose a letter (in some cases several letters) all the coin's public addresses will start with. It is defined by CRYPTONOTE_PUBLIC_ADDRESS_BASE58_PREFIX constant. Since the rules for address prefixes are nontrivial you may use the [prefix generator tool]

const uint64_t CRYPTONOTE_PUBLIC_ADDRESS_BASE58_PREFIX = 0xe9; // addresses start with "f"
Sixth step. Genesis block
1. Build the binaries with blank genesis tx hex

You should leave const char GENESIS_COINBASE_TX_HEX[] blank and compile the binaries without it.

const char GENESIS_COINBASE_TX_HEX[] = "Nuestro primer bloque genesis de nuestra propia Blockchain";
2. Start the daemon to print out the genesis block

Run your daemon with --print-genesis-tx argument. It will print out the genesis block coinbase transaction hash.

furiouscoind --print-genesis-tx
3. Copy the printed transaction hash

Copy the tx hash that has been printed by the daemon to GENESIS_COINBASE_TX_HEX in src/CryptoNoteConfig.h

const char GENESIS_COINBASE_TX_HEX[] = "013c01ff0001ffff...785a33d9ebdba68b0";
4. Recompile the binaries

Recompile everything again. The coin code is ready now.

Building
On *nix
Dependencies: GCC 4.7.3 or later, CMake 2.8.6 or later, and Boost 1.55.

You may download them from:

http://gcc.gnu.org/
http://www.cmake.org/
http://www.boost.org/
Alternatively, it may be possible to install them using a package manager.
To build, change to a directory where this file is located, and run make. The resulting executables can be found in build/release/src.

Advanced options:

Parallel build: run make -j<number of threads> instead of make.
Debug build: run make build-debug.
Test suite: run make test-release to run tests in addition to building. Running make test-debug will do the same to the debug version.
Building with Clang: it may be possible to use Clang instead of GCC, but this may not work everywhere. To build, run export CC=clang CXX=clang++ before running make.
On Windows
Dependencies: MSVC 2013 or later, CMake 2.8.6 or later, and Boost 1.55. You may download them from:

http://www.microsoft.com/
http://www.cmake.org/
http://www.boost.org/
To build, change to a directory where this file is located, and run theas commands:

mkdir build
cd build
cmake -G "Visual Studio 12 Win64" ..
And then do Build.
