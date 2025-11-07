

#include <bits/stdc++.h>
using namespace std;

class Player {
public:
    string name;
    bool alive;
    Player* target;

    Player(string n) : name(n), alive(true), target(nullptr) {}

    void eliminate() {
        alive = false;
        cout << "💀 " << name << " has been eliminated!\n";
    }
};

class AssassinGame {
    vector<Player*> players;
    mt19937 rng{random_device{}()};

public:
    AssassinGame(const vector<string>& names) {
        for (auto& n : names)
            players.push_back(new Player(n));
        assignTargets();
    }

    ~AssassinGame() {
        for (auto* p : players)
            delete p;
    }

    void assignTargets() {
        vector<int> idx(players.size());
        iota(idx.begin(), idx.end(), 0);
        shuffle(idx.begin(), idx.end(), rng);

        for (size_t i = 0; i < idx.size(); ++i) {
            players[idx[i]]->target = players[idx[(i + 1) % idx.size()]];
        }

        cout << "🎯 Targets Assigned!\n";
        for (auto* p : players)
            cout << "• " << p->name << " → " << p->target->name << "\n";
        cout << "\n";
    }

    void simulate() {
        cout << "🔪 Let the assassination begin!\n\n";
        int round = 1;

        while (aliveCount() > 1) {
            cout << "===== ROUND " << round++ << " =====\n";
            vector<Player*> alivePlayers = getAlivePlayers();
            shuffle(alivePlayers.begin(), alivePlayers.end(), rng);

            for (auto* assassin : alivePlayers) {
                if (!assassin->alive) continue;
                Player* target = assassin->target;

                if (target && target->alive) {
                    // Random chance of success
                    if (rand() % 100 < 60) {
                        cout << "🗡️  " << assassin->name << " assassinates " << target->name << "!\n";
                        target->eliminate();

                        // transfer target
                        assassin->target = target->target;
                    } else {
                        cout << "❌ " << assassin->name << " failed to eliminate " << target->name << "!\n";
                    }
                }
            }

            cout << "\nRemaining assassins: " << aliveCount() << "\n\n";
        }

        cout << "🏆 Winner: " << getAlivePlayers()[0]->name << " — the ultimate assassin!\n";
    }

private:
    int aliveCount() const {
        return count_if(players.begin(), players.end(), [](Player* p){ return p->alive; });
    }

    vector<Player*> getAlivePlayers() const {
        vector<Player*> res;
        for (auto* p : players)
            if (p->alive) res.push_back(p);
        return res;
    }
};

int main() {
    srand(time(0));
    vector<string> names = {"Sangram", "Aarav", "Mira", "Karan", "Zoya"};
    AssassinGame game(names);
    game.simulate();
    return 0;
}
