# CplusplusBlcochain
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
