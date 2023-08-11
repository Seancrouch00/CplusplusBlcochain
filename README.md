# CplusplusBlcochain
Blockchain:
Creating a complete blockchain in C++ involves several components such as data structures, cryptographic operations, and network communication. Here's a simplified example of how you might structure the code for a basic blockchain:

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <ctime>
#include <sstream>
#include <openssl/sha.h>

class Block {
public:
    int index;
    std::string previousHash;
    std::string timestamp;
    std::string data;
    std::string hash;

    Block(int idx, const std::string& prevHash, const std::string& data) : index(idx), previousHash(prevHash), data(data) {
        timestamp = getCurrentTimestamp();
        hash = calculateHash();
    }

private:
    std::string getCurrentTimestamp() {
        std::time_t now = std::time(nullptr);
        return std::asctime(std::localtime(&now));
    }

    std::string calculateHash() {
        std::stringstream ss;
        ss << index << previousHash << timestamp << data;
        std::string input = ss.str();

        unsigned char hash[SHA256_DIGEST_LENGTH];
        SHA256(reinterpret_cast<const unsigned char*>(input.c_str()), input.length(), hash);

        std::stringstream hashStream;
        for (int i = 0; i < SHA256_DIGEST_LENGTH; ++i) {
            hashStream << std::hex << static_cast<int>(hash[i]);
        }

        return hashStream.str();
    }
};

class Blockchain {
private:
    std::vector<Block> chain;

public:
    Blockchain() {
        chain.emplace_back(Block(0, "0", "Genesis Block"));
    }

    void addBlock(const std::string& data) {
        int index = chain.size();
        const std::string& prevHash = chain[index - 1].hash;
        chain.emplace_back(Block(index, prevHash, data));
    }
};

int main() {
    Blockchain blockchain;

    blockchain.addBlock("Transaction 1");
    blockchain.addBlock("Transaction 2");

    // Display blockchain contents
    for (const Block& block : blockchain.chain) {
        std::cout << "Block #" << block.index << "\n";
        std::cout << "Timestamp: " << block.timestamp;
        std::cout << "Previous Hash: " << block.previousHash << "\n";
        std::cout << "Data: " << block.data << "\n";
        std::cout << "Hash: " << block.hash << "\n\n";
    }

    return 0;
}
```

Please note that this is a simplified example for educational purposes and lacks many real-world considerations like networking, consensus algorithms, and security measures. If you're serious about developing a blockchain, it's essential to delve into more advanced topics and possibly collaborate with experts in the field.

SMART CONTRACT:
here's a simple example of a C++ smart contract using the EOSIO blockchain platform. This contract represents a basic token:

```cpp
#include <eosio/eosio.hpp>

class [[eosio::contract]] token_contract : public eosio::contract {
public:
    using eosio::contract::contract;

    [[eosio::action]]
    void create(eosio::name issuer, eosio::asset maximum_supply) {
        require_auth(_self);

        eosio::check(is_account(issuer), "Issuer account does not exist");

        auto symbol = maximum_supply.symbol;
        eosio::check(symbol.is_valid(), "Invalid symbol name");
        eosio::check(maximum_supply.is_valid(), "Invalid supply");
        eosio::check(maximum_supply.amount > 0, "Max supply must be positive");

        stats statstable(_self, symbol.code().raw());
        auto existing = statstable.find(symbol.code().raw());
        eosio::check(existing == statstable.end(), "Token with symbol already exists");

        statstable.emplace(_self, [&](auto& s) {
            s.supply.symbol = maximum_supply.symbol;
            s.max_supply = maximum_supply;
            s.issuer = issuer;
        });
    }

    struct [[eosio::table]] stoken {
        eosio::asset supply;
        eosio::asset max_supply;
        eosio::name issuer;

        uint64_t primary_key() const { return supply.symbol.code().raw(); }
    };

    using stats = eosio::multi_index<"stats"_n, stoken>;

private:
    void sub_balance(eosio::name owner, eosio::asset value) {
        accounts from_acnts(_self, owner.value);

        auto from = from_acnts.find(value.symbol.code().raw());
        eosio::check(from != from_acnts.end(), "Account does not have enough balance");
        eosio::check(from->balance.amount >= value.amount, "Overdrawn balance");

        from_acnts.modify(from, _self, [&](auto& a) {
            a.balance -= value;
        });
    }

    void add_balance(eosio::name owner, eosio::asset value, eosio::name ram_payer) {
        accounts to_acnts(_self, owner.value);
        auto to = to_acnts.find(value.symbol.code().raw());
        if (to == to_acnts.end()) {
            to_acnts.emplace(ram_payer, [&](auto& a) {
                a.balance = value;
            });
        } else {
            to_acnts.modify(to, eosio::same_payer, [&](auto& a) {
                a.balance += value;
            });
        }
    }

    struct [[eosio::table]] account {
        eosio::asset balance;

        uint64_t primary_key() const { return balance.symbol.code().raw(); }
    };

    using accounts = eosio::multi_index<"accounts"_n, account>;

};

EOSIO_DISPATCH(token_contract, (create))
```

Please note that this is a very basic example and may need to be customized according to your specific needs. Also, remember to adapt the code to the blockchain platform you are working with, as different platforms might have variations in syntax and features.
