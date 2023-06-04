# Beyond Blockchain: Hashgraph Hackathon 2023The code above provides a solution utilizing web3 to help provide a safe means of sending updates from the OEM to thethe vehicles directly. Below is an in-depth analysis of the idea to help provide context and further elaborate on thesolution. Take a look at the README.md to learn more about problem space, how the code above helps to solve it, and ademo.## ProblemModern cars are rapidly embracing autonomous and eco-friendly functionalities like adaptive cruise control, lane-keepingassist, and complete autonomous driving.The <a href="https://www.weforum.org/agenda/2018/01/8-ways-ai-can-help-save-the-planet/">World Economic Forum</a>recognizes that implementing route optimizations, eco-driving algorithms, and introducing autonomous driving ride-shareservices can play a significant role in curbing greenhouse gas emissions and protecting the environment.However, the advent of these advanced technologies also expands the potential attack surface for cyber attackers toexploit. Of particular concern is the vulnerability during the software/firmware update process, which often becomesa prime target for cyberattacks. As the prevalence of autonomous features grows, the frequency of updates aimed atenhancing and securing these systems will inevitably increase.## SolutionThe solution proposed to address the aforementioned problem involves a collaborative effort between the OEM (OriginalEquipment Manufacturer) and the vehicles themselves. This solution leverages the secure capabilities of Hedera Hashgraphand utilizes IPFS (InterPlanetary File System) to ensure the safe transmission and storage of crucial information duringsoftware/firmware updates.The solution is divided into two key components, each making use of the unique strengths offered by the respectivetechnologies. IPFS, designed for decentralized storage, facilitates the convenient storage of encrypted update files. Onthe other hand, Hedera Hashgraph, known for its secure and swift transactions, enables the secure transmission ofsensitive data such as checksums and encryption keys.By combining the strengths of IPFS and Hedera Hashgraph, this solution provides a robust and trustworthy mechanism forhandling software/firmware updates in autonomous vehicles, addressing the vulnerabilities associated with the largerattack surface introduced by these advanced technologies.```mermaid  graph LR;    A[ OEM ] -- Secret Update Info --> B[ Hashgraph ];    A[ OEM ] -- Encrypted Update --> C[ IPFS ];    B[ Hashgraph ] -- Secret Update Info --> D[ Vehicle ];    C[ IPFS ] -- Encrypted Update --> D[ Vehicle ];```## Demo## How to Run### Prerequisites### Running the Code## Future Work