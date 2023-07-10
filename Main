def get_todays_metadata():
    import requests, json
    r = requests.get("https://www.nytimes.com/puzzles/letter-boxed")
    start_string = r.text.index("window.gameData")
    start_parens = start_string + r.text[start_string:].index("{")
    end_string = start_parens+ r.text[start_parens:].index(",\"dictionary")
    todays_metadata = json.loads(r.text[start_parens:end_string]+"}")
    return {'sides': todays_metadata['sides'], 'nyt_solution': todays_metadata['ourSolution']}


class TrieNode:
    def __init__(self):
        self.children = {}
        self.end_of_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for letter in word:
            if letter not in node.children:
                node.children[letter] = TrieNode()
            node = node.children[letter]
        node.end_of_word = True

    def search_prefix(self, prefix):
        node = self.root
        for letter in prefix:
            if letter not in node.children:
                return None
            node = node.children[letter]
        return node


def build_graph(puzzle):
    graph = {letter: set() for letter in puzzle}
    for i, letter in enumerate(puzzle):
        for j in range(len(puzzle)):
            if j // 3 != i // 3:
                graph[letter].add(puzzle[j])
    return graph

def load_dictionary(filename, puzzle):
    trie = Trie()
    with open(filename, 'r') as f:
        for word in f:
            word = word.strip().lower()
            if set(word).issubset(set(puzzle)):
                trie.insert(word)
    return trie

def find_potential_words(graph, trie, start, path, potential_words):
    current_word = ''.join(path)
    node = trie.search_prefix(current_word)
    if node:
        if node.end_of_word:
            potential_words.add(current_word)
            
        for letter in graph[start]:
            find_potential_words(graph, trie, letter, path + [letter], potential_words)

def build_word_graph(potential_words):
    word_graph = {word: [] for word in potential_words}
    for word in potential_words:
        for next_word in potential_words:
            if word[-1] == next_word[0] and word != next_word:
                word_graph[word].append(next_word)
    return word_graph

def find_chains(word_graph, max_chain_length):
    dp = {length: {word: [] for word in word_graph} for length in range(1, max_chain_length + 1)}
    
    for word in word_graph:
        dp[1][word] = [[word]]
        
    for length in range(2, max_chain_length + 1):
        for word in word_graph:
            for next_word in word_graph[word]:
                for chain in dp[length - 1][next_word]:
                    dp[length][word].append([word] + chain)
                    
    chains = []
    for length_chains in dp.values():
        for word_chains in length_chains.values():
            chains.extend(word_chains)
            
    return chains


def valid_move(puzzle, word):
    from collections import Counter

    # Check that we are not reusing the same side of the box immediately
    for i in range(len(word) - 1):
        last_pos = puzzle.index(word[i])
        next_pos = puzzle.index(word[i+1])
        if last_pos // 3 == next_pos // 3:
            return False

    # Check that the puzzle has enough of each letter
    puzzle_letter_counts = Counter(puzzle)
    word_letter_counts = Counter(word)
    for letter, count in word_letter_counts.items():
        if count > puzzle_letter_counts[letter]:
            return False

    return True


def uses_all_letters(chain, puzzle):
    return set(''.join(chain)) == set(puzzle)

def main():
    #puzzle = input("Enter the letters in the puzzle in clockwise order starting from the top left letter: ").lower()
    #puzzle = "trqeompdusan"
    todays_metadata = get_todays_metadata() # get today's puzzle data
    puzzle = ''.join(todays_metadata['sides']).lower() # convert sides to puzzle string
    graph = build_graph(puzzle)
    trie = load_dictionary('/Users/nachumsash/Documents/Anagram Files/cleanedNormalWords3.txt', puzzle)
    potential_words = set()
    for letter in graph:
        find_potential_words(graph, trie, letter, [letter], potential_words)
    print(potential_words)
    chains = set()
    # In your main function, replace the chain finding part with:
    word_graph = build_word_graph(potential_words)
    chains = find_chains(word_graph, 3)
    for chain in chains:
        if uses_all_letters(chain, puzzle):
            print(" -> ".join(chain))



if __name__ == "__main__":
    main()
