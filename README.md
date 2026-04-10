# ---------- NFA DEFINITION ----------
nfa = {
    'q0': {'a': ['q0', 'q1'], '': ['q2']},
    'q1': {'b': ['q2']},
    'q2': {'a': ['q2'], 'b': ['q1']}
}

alphabet = ['a', 'b']
start_state = 'q0'
final_states = {'q2'}


# ---------- ε-CLOSURE ----------
def epsilon_closure(states, nfa):
    closure = set(states)
    stack = list(states)

    while stack:
        state = stack.pop()
        for next_state in nfa.get(state, {}).get('', []):
            if next_state not in closure:
                closure.add(next_state)
                stack.append(next_state)

    return closure


# ---------- MOVE FUNCTION ----------
def move(states, symbol, nfa):
    result = set()
    for state in states:
        result.update(nfa.get(state, {}).get(symbol, []))
    return result


# ---------- NFA TO DFA ----------
def nfa_to_dfa(nfa, start_state, alphabet):
    start = frozenset(epsilon_closure({start_state}, nfa))

    dfa_states = [start]
    unmarked = [start]
    dfa_transitions = {}

    while unmarked:
        current = unmarked.pop()

        for symbol in alphabet:
            next_states = move(current, symbol, nfa)
            closure = frozenset(epsilon_closure(next_states, nfa))

            dfa_transitions[(current, symbol)] = closure

            if closure not in dfa_states:
                dfa_states.append(closure)
                unmarked.append(closure)

    return dfa_states, dfa_transitions


# ---------- DFA FINAL STATES ----------
def get_dfa_final_states(dfa_states, nfa_final):
    dfa_final = set()
    for state in dfa_states:
        if any(s in nfa_final for s in state):
            dfa_final.add(state)
    return dfa_final


# ---------- DFA MINIMIZATION ----------
def minimize_dfa(dfa_states, dfa_transitions, final_states, alphabet):

    P = [set(final_states), set(dfa_states) - set(final_states)]
    W = [set(final_states)]

    while W:
        A = W.pop()

        for c in alphabet:
            X = set()

            for (state, symbol), target in dfa_transitions.items():
                if symbol == c and target in A:
                    X.add(state)

            new_P = []
            for Y in P:
                inter = Y & X
                diff = Y - X

                if inter and diff:
                    new_P.append(inter)
                    new_P.append(diff)

                    if Y in W:
                        W.remove(Y)
                        W.append(inter)
                        W.append(diff)
                    else:
                        if len(inter) <= len(diff):
                            W.append(inter)
                        else:
                            W.append(diff)
                else:
                    new_P.append(Y)

            P = new_P

    return P


# ---------- MAIN EXECUTION ----------
dfa_states, dfa_transitions = nfa_to_dfa(nfa, start_state, alphabet)
dfa_final_states = get_dfa_final_states(dfa_states, final_states)
minimized_partitions = minimize_dfa(dfa_states, dfa_transitions, dfa_final_states, alphabet)


# ---------- OUTPUT ----------
print("\nDFA STATES:")
for s in dfa_states:
    print(s)

print("\nDFA TRANSITIONS:")
for (state, symbol), target in dfa_transitions.items():
    print(f"{state} --{symbol}--> {target}")

print("\nDFA FINAL STATES:")
for s in dfa_final_states:
    print(s)

print("\nMINIMIZED DFA PARTITIONS:")
for group in minimized_partitions:
    print(group)
