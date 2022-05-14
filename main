import array as arr
import pandas
from options import *
from pandas import *
import itertools

class Main:
    def __init__(self):
        self.running = True
        self.choice = 1
        self.df = DataFrame()   # Basic Table
        self.df_complete_deter = DataFrame()   # Complete Table
        self.all_states = []
        self.display_all_automata()
        while self.running:
            self.menu()

    def menu(self):
        self.choice = int(input("Which automaton do you want to use : "))
        while len(All_automaton) < self.choice or self.choice < 1 or self.choice is None:
            self.display_all_automata()
            self.choice = int(input("No such file in the database !\nWhich automaton do you want to use : "))
        print("/////////////////////////////////////////////////////////////////////////////////////////\n")
        print("Execute automaton n°", self.choice, ":\n")
        file = All_automaton[self.choice - 1]
        malloc = self.malloc_table(file)
        fin = self.create_table(file, malloc)

        if self.check_synchro(file) != "Synchronous":  ### If it is asynchronous
            print("Asynchronous Automaton, we can't synchronize it yet...")
            self.display_table(file, fin)
            print("\nBut we can complete it !")
            print(self.completion_asynchronous(final_table=fin, NameFile=file))

        else:  ### If it is synchronous
            print("Synchronous Automaton\n")
            self.df = self.display_table(file, fin)

            if len(self.check_complete(fin)) == 0:
                print("Automaton n°", self.choice, "is already complete !")

            else:
                check = self.check_deter(nb_init_stat=self.get_initial_states(file)[0][0], NameFile=file)
                if check != "Deterministic":
                    print("Automaton not deterministic !\nHere is the reasons :", check)
                    self.df_complete_deter = self.determinization_synchronous(fin)
                    self.df = self.df_complete_deter[0]
                    print("Automaton n°", self.choice, "deterministic but not complete yet!\n\n",DataFrame(self.df))
                self.all_states = [str(x) for x in self.df.index]
                fin = list(self.df.values.tolist())
                fin[0] = [[x] for x in fin[0]]
                self.df = self.completion_synchronous(file, fin, self.all_states)[0]
            fin = list(self.df.values.tolist())
            isempty = self.df_complete_deter[0].empty
            if not isempty:
                check_T_NT = self.check_term_stat(terminal_stat=self.df_complete_deter[2], CDFA=fin)
            new_states = self.df_complete_deter[3]
            if len(self.check_complete(fin)) == 0: new_states.append("p")
            T_NT_states = self.T_NT_groups(new_states)
            print("Terminal and Non Terminal states groups : \n", T_NT_states)
            print(DataFrame(check_T_NT, index=self.df.index, columns=self.df.columns), "\n")
            nb_combination_T_NT = self.get_nb_symbols(file) ** 2
            print("T_NT pattern :", self.T_NT_pattern(nb_states=self.get_nb_symbols(file)))

        self.df = DataFrame()

    def get_nb_states(self, NameFile):  # get the number of states (initial included)
        file = open(NameFile, "r")
        line = ""
        for i in range(2):
            line = file.readline()
        return int(line)

    def get_nb_symbols(self, NameFile):  # get the number of alphabet a,b,c ==> 3
        file = open(NameFile, "r")
        line = ""
        nb_symbols = int(file.readline())
        epsilon = False
        for i in range(6):
            line = file.readline()
        while line != "" and epsilon == False:
            if "*" in line: epsilon = True
            line = file.readline()
        if epsilon: return nb_symbols+1
        return nb_symbols

    def get_initial_states(self, NameFile):  # index 0 : Nb of initial states ==> 3 | index 1 : Actual initial states ==> a,b,c
        file = open(NameFile, "r")
        line = ""
        for i in range(3):
            line = file.readline()
        line_split = line.split(" ")
        nb_init = [int(line_split[0])]
        all_init = []
        del line_split[0]
        for item in line_split:
            all_init.append(int(item))
        final = [nb_init, all_init]
        return final

    def get_final_states(self, NameFile):  # index 0 : Nb of final states ==> 3 | index 1 : Actual final states ==> a,b,c
        file = open(NameFile, "r")
        line = ""
        for i in range(4):
            line = file.readline()
        line_split = line.split(" ")
        nb_init = [int(line_split[0])]
        all_init = []
        del line_split[0]
        for item in line_split:
            all_init.append(int(item))
        final = [nb_init, all_init]
        return final

    def get_nb_transition(self, NameFile):  # get number of transitions
        file = open(NameFile, "r")
        line = ""
        for i in range(5):
            line = file.readline()
        return int(line)

    def all_transitions(self, NameFile):  # gives an array of transitions in the form : <base state><symbol><final state>
        file = open(NameFile, "r")
        line = file.readline()
        all_transi = ""
        for i in range(5):
            line = file.readline()
        while line != "":
            all_transi += line
            line = file.readline()
        return all_transi.split()

    def get_states(self, NameFile):
        file = open(NameFile, "r")
        line = ""
        all_states = []
        for i in range(6):
            line = file.readline()
        while line != "":
            if line[0] not in all_states: all_states.append(line[0])
            line = file.readline()
        return all_states

    def nb_symbols_to_symbols(self, NameFile, nb_symbols):  # gives an array of transition depending on its number
        symbols = []
        i = 0
        while nb_symbols > 0:
            symbols.append(chr(ord("a") + i))
            i += 1
            nb_symbols -= 1
        if self.check_synchro(NameFile) != "Synchronous":
            symbols[i-1] = "*"
        return symbols

    def malloc_table(self, NameFile):  # allocate some space to create a new table
        nb_states = self.get_nb_states(NameFile)
        nb_symbols = self.get_nb_symbols(NameFile)
        final_table = [[[] for x in range(nb_symbols)] for y in range(nb_states)]
        return final_table

    def create_table(self, NameFile, final_table):
        file = open(NameFile, "r")
        line = ""
        for z in range(6):
            line = file.readline()

        nb_states = self.get_nb_states(NameFile)
        nb_symbols = self.get_nb_symbols(NameFile)
        if self.check_synchro(NameFile) != "Synchronous":
            for i in range(nb_states):  # State
                for j in range(nb_symbols):  # Letter
                    var_letter = self.nb_symbols_to_symbols(NameFile, nb_symbols)  # ==> a, b, c,...
                    var_letter.append("*")
                    x = var_letter[j]  # ==> a, b, c, d, * ...
                    while line != "":
                        if line.startswith(str(i) + str(x)):
                            final_table[i][j].append(line[2])
                        line = file.readline()
                    file.close()
                    file = open(NameFile, "r")
                    line = ""
                    for z in range(6):
                        line = file.readline()
        else:
            for i in range(nb_states):  # State
                for j in range(nb_symbols):  # Letter
                    var_letter = self.nb_symbols_to_symbols(NameFile, nb_symbols)  # ==> a, b, c,...
                    x = var_letter[j]  # ==> a, b, c, d, * ...
                    while line != "":
                        if line.startswith(str(i) + str(x)):
                            final_table[i][j].append(line[2])
                        line = file.readline()
                    file.close()
                    file = open(NameFile, "r")
                    line = ""
                    for z in range(6):
                        line = file.readline()
        return final_table

    def display_table(self, NameFile, final_table):
        symbols = self.nb_symbols_to_symbols(NameFile, self.get_nb_symbols(NameFile))
        print("ALPHABET :", symbols)
        all_states = []
        nb_states = self.get_nb_states(NameFile)
        print("INITIAL STATES :", self.get_initial_states(NameFile))
        print("FINAL STATES :", self.get_final_states(NameFile), "\n")
        for i in range(nb_states):
            if i in self.get_initial_states(NameFile)[1] and i in self.get_final_states(NameFile)[1]:
                all_states.append("<=> " + str(i))
            elif i in self.get_initial_states(NameFile)[1]:
                all_states.append("--> " + str(i))
            elif i in self.get_final_states(NameFile)[1]:
                all_states.append("<-- " + str(i))
            else:
                all_states.append("    " + str(i))
        df = pandas.DataFrame(final_table, columns=symbols, index=all_states)
        print(DataFrame(df))
        print("/////////////////////////////////////////////////////////////////////////////////////////\n")
        return DataFrame(df)

    def check_synchro(self, NameFile):
        Arr = []

        file = open(NameFile, "r")
        line = ""
        for z in range(6):
            line = file.readline()

        while line != "":  ### We add all transition with the empty word to a list
            if "*" in line:  ### It will allow to say where the automaton is asynchronous and if the list is empty that it's synchronous
                Arr.append(line[0:3])
            line = file.readline()
        if len(Arr) == 0:
            return "Synchronous"
        return Arr

    def check_deter(self, nb_init_stat, NameFile):  ### fonction to test if it's deterministic or not
        if nb_init_stat > 1:  ### we check the number of entrie
            return "multiple initial states"
        else:
            Arr = []

            file = open(NameFile, "r")
            line = ""
            for z in range(6):
                line = file.readline()

            while line != "":  ### those boocle are here to compare our different transitions and check
                Arr.append(str(line[0:2]))  ### if there is from a same state transition multiple ssame transition
                line = file.readline()  ### Example: forme state 1 multiple arrow 'a'
            reason_false = []
            for i in range(len(Arr)):
                for j in range(i + 1, len(Arr)):
                    if Arr[i] == Arr[j]:
                        reason_false.append(Arr[i])
            if len(reason_false) == 0:
                return "Deterministic"

            return reason_false

    def check_complete(self, Final_table):
        Arr = []
        for i in range(len(Final_table)):  ### We put in an array where the is no transition if the array is null it's complete
            for j in range(len(Final_table[i])):
                if len(Final_table[i][j]) == 0 or Final_table[i][j] ==[""]:
                    Arr.append([i, j])
        return Arr

    def completion_synchronous(self, NameFile, incomplete_table, all_states):
        empty = self.check_complete(incomplete_table)
        for i in range(len(empty)):
            coord = empty[i]
            incomplete_table[coord[0]][coord[1]] = "p"
        incomplete_table.append([])
        symbols = self.nb_symbols_to_symbols(NameFile, self.get_nb_symbols(NameFile))
        # symbols.append("*")
        nb_states = all_states.__len__()
        nb_symbols = self.get_nb_symbols(NameFile)
        all_states.append("p")
        last = len(incomplete_table) - 1
        for i in range(nb_symbols):
            incomplete_table[last].append("")
        for j in range(nb_symbols):
            incomplete_table[last][j] += 'p'
            j += 1
        df = DataFrame(incomplete_table, columns=symbols, index=all_states)
        print("\nAutomaton n°", self.choice, "completed and deterministic !\n")
        return self.display_dataFrame_determinization(df)

    def completion_asynchronous(self, NameFile, final_table):
        empty = self.check_complete(final_table)
        for i in range(len(empty)):
            coord = empty[i]
            final_table[coord[0]][coord[1]] = "p"
        final_table.append([])
        symbols = self.nb_symbols_to_symbols(nb_symbols=self.get_nb_symbols(NameFile), NameFile=All_automaton[self.choice-1])
        all_states = []
        #print(symbols)
        nb_states = self.get_nb_states(NameFile)
        nb_symbols = self.get_nb_symbols(NameFile)
        for i in range(nb_states):
            if i in self.get_initial_states(NameFile)[1] and i in self.get_final_states(NameFile)[1] : all_states.append("<=> "+str(i))
            elif i in self.get_initial_states(NameFile)[1] : all_states.append("--> "+str(i))
            elif i in self.get_final_states(NameFile)[1]: all_states.append("<-- "+str(i))
            else: all_states.append("    " + str(i))
        all_states.append("    p")
        last = len(final_table) - 1
        for i in range(nb_symbols):
            final_table[last].append("")
        for j in range(nb_symbols):
            final_table[last][j] += 'p'
            j += 1
        df = pandas.DataFrame(final_table, columns=symbols, index=all_states)
        print("initial :", self.get_initial_states(NameFile))
        print("final :", self.get_final_states(NameFile))
        return DataFrame(df)

    def add_automata(self):
        name = input("Give me the name of your new automata (with the extension): ")
        All_automaton.append(name)
        print("The file :" + name + " is at position", All_automaton.__len__())

    def display_all_automata(self):
        i = 1
        print("All Files : ")
        for auto in All_automaton:
            print(str(i) + " : " + auto)
            i += 1
        print("\n")

    print("/////////////////////////////////////////////////////////////////////////////////////////\n")

    def array_to_dict(self, final_table):  # converts an array into a dictionary ,easier for our next calculations
        dict = {}
        return_dict = {}
        all_states = self.get_states(All_automaton[self.choice-1])
        if len(self.check_complete(final_table)) != 0: all_states.append("p")
        #print("ALL STATES :", all_states)
        j = 0
        nb_symbols = self.get_nb_symbols(All_automaton[self.choice-1])
        all_symbols = self.nb_symbols_to_symbols(All_automaton[self.choice-1], nb_symbols)
        for i in final_table:
            dict[all_states[j]] = i
            j += 1

        k = 0  # Letters
        for d in list(dict):
            return_dict[d] = {}
            for a in all_symbols:
                for i in range(nb_symbols):
                    return_dict[d][a] = dict[d][k]
                k += 1
            k = 0
        return return_dict

    def reversed_states(self, string):  # to avoid redundancy | for example : key_list = ['13', '31']  ==> '13' and '31' are redundant
        return string [::-1]

    def string_to_list(self, string):
        list1 = []
        list1[:0] = string
        return list1

    def check_not_in(self, new_states, keys):  #  Avoid redundant new states
        Arr = []
        all_combinasion = list(itertools.permutations (new_states, len(new_states)))
        for a in all_combinasion:
            Arr.append("".join(a))
        for a in Arr:
            if a in keys: return False
        return True

    def determinization_synchronous(self, final_table):
        nfa_dict = self.array_to_dict(final_table)
        final_states = self.get_final_states(All_automaton[self.choice-1])
        new_states_list = []
        dfa = {}
        #keys_list = self.get_states(All_automaton[self.choice-1])
        keys_list = list(list(nfa_dict.keys())[0])
        nb_symbols = self.get_nb_symbols(All_automaton[self.choice-1])
        symbols_list = self.nb_symbols_to_symbols(All_automaton[self.choice-1], nb_symbols)

        #print("SYMBOLS LIST :", symbols_list)
        #print("KEYS LIST :", keys_list)
        dfa[keys_list[0]] = {}
        for i in range(nb_symbols):
            var = "".join(nfa_dict[keys_list[0]][symbols_list[i]])  # creating a single string from all the elements of the list which is a new state
            dfa[keys_list[0]][symbols_list[i]] = var  # assigning the state in DFA table
            if var not in keys_list and var != "":  # if the state is newly created
                new_states_list.append(var)  # then append it to the new_states_list
                keys_list.append(var)
            if var != "":
                new_states_list.append(var)

        while len(new_states_list) != 0:  # condition is true only if the new_states_list is not empty
            dfa[new_states_list[0]] = {}  # taking the first element of the new_states_list and examining it
            for p in range(len(new_states_list[0])):
                for i in range(len(symbols_list)):
                    temp = []  # creating a temporay list
                    for j in range(len(new_states_list[0])):
                        temp += nfa_dict[new_states_list[0][j]][symbols_list[i]]  # taking the union of the states
                    s = ""
                    s = s.join(temp)  # creating a single string(new state) from all the elements of the list
                    s = "".join(dict.fromkeys(s))
                    not_in = self.check_not_in(s, keys_list)
                    if not_in and s != "":  # if the state is newly created
                        new_states_list.append(s)  # then append it to the new_states_list
                        keys_list.append(s)  # as well as to the keys_list which contains all the states
                    dfa[new_states_list[0]][symbols_list[i]] = [s]  # assigning the new state in the DFA table
            new_states_list.remove(new_states_list[0])

        all_old_initial_states = self.get_initial_states(All_automaton[self.choice - 1])[1]
        dataF_return_dict = DataFrame(dfa)
        return_dfa = dataF_return_dict.transpose()
        new_init = self.I_new_initial_state(All_automaton[self.choice - 1], self.array_to_dict(self.df.values), all_old_initial_states)
        init_df = DataFrame(new_init[0])
        init_df = init_df.transpose()
        test = return_dfa.to_dict()
        test = DataFrame(test).transpose().to_dict()
        i = 0
        for a in new_states_list:
            test.pop(a, None)
            new_states_list.pop(i)
            i += 1
        test = DataFrame(test).transpose().to_dict()
        df = [DataFrame(init_df.to_dict()), DataFrame(test)]
        df = DataFrame(concat(df))

        new_states_list.insert(0, new_init[1])
        return self.display_dataFrame_determinization(DataFrame(return_dfa))
        #############################

    def display_dataFrame_determinization(self, df):
        symbols = self.nb_symbols_to_symbols(All_automaton[self.choice-1], self.get_nb_symbols(All_automaton[self.choice-1]))  # Get the alphabet of the FA
        all_old_initial_states = self.get_initial_states(All_automaton[self.choice-1])[1]  #  Get the initial states of the FA
        all_old_final_states = self.get_final_states(All_automaton[self.choice-1])[1]  #  Get the final states of the FA
        all_old_initial_states = [str(x) for x in all_old_initial_states]   #  Converting the initial states of the FA into strings
        all_old_final_states = [str(x) for x in all_old_final_states]  #  Converting the final states of the FA into strings
        all_new_states = [str(x) for x in df.index]  #  Converting the new states of the FA into strings
        print("///// RECAP /////\nALL NEW STATES :", all_new_states)  # Display the new states
        print("ALPHABET :",symbols,"\nINITIAL STATES :", all_old_initial_states, "\nFINAL STATES :", all_old_final_states)
        arrow_states = []   #  The state's columns with the correct arrows
        normal_states = []   #  The state's columns without the arrows
        new_final_states = []
        new_initial_states = []


        ######### JE VEUX ICI CREER UNE NOUVELLE STATES COMPOSER DE INITIAL STATE ET SUPPRIMER LES ANCIENNES STATES QUI LA COMPOSE #######

        """new_init = self.I_new_initial_state(All_automaton[self.choice-1], self.array_to_dict(self.df.values), all_old_initial_states)
        init_df = DataFrame(new_init[0])
        init_df = init_df.transpose()
        test = df.to_dict()
        test = DataFrame(test).transpose().to_dict()
        i = 0
        for a in all_new_states:
            test.pop(a, None)
            all_new_states.pop(i)
            i += 1
        test = DataFrame(test).transpose().to_dict()
        df = [DataFrame(init_df.to_dict()), DataFrame(test)]
        df = DataFrame(concat(df))

        all_new_states.insert(0, new_init[1])"""
        arrow = ""
        for a in all_new_states:
            check = False  # to know if an arrow has already been assigned for a give state
            for i in range(len(a)):
                if a[i] in all_old_final_states and a[i] in all_old_initial_states:  #  If the state belongs to the initial and final states
                    check = True
                    arrow = "<-- "
                elif a[i] in all_old_final_states:   #  If the state belongs to the final states
                    check = True
                    if arrow == "--> ": arrow = "<=> "
                    else: arrow = "<-- "
                elif a[i] in all_old_initial_states:   #  If the state belongs to the initial states
                    check = True
                    if arrow == "<-- ": arrow = "<=> "
                    else: arrow = "--> "
                elif not check : arrow = "    "   #  Else we import nothing to this columns, just some spacebars
            arrow_states.append(arrow + str(a))
            normal_states.append(str(a))
            if arrow == "<-- " or arrow == "<=> ": new_final_states.append(str(a))
            elif arrow == "--> " : new_initial_states.append(str(a))
            arrow = ""
            check = False
        print(DataFrame(df.values, index=arrow_states, columns=symbols))
        print("/////////////////////////////////////////////////////////////////////////////////////////\n")
        return DataFrame(df.values, index=normal_states, columns=symbols), new_initial_states, new_final_states, arrow_states

    def I_new_initial_state(self, NameFile, table, initial_states):
        global var
        nb_symbols = self.get_nb_symbols(NameFile)
        symbols = self.nb_symbols_to_symbols(NameFile, nb_symbols)
        arr = list(table.values())
        concat = {}
        for i in range(len(arr)):
            if str(i) in initial_states:
                concat[i] = arr[i]
        concat_initial_dict = {}
        for symb in symbols:
            concat_initial_dict[symb] = []
        for c in concat.values():
            for symb in symbols:
                concat_initial_dict[symb].append(c[symb])

        arr = []
        i = 0
        for a in concat_initial_dict:
            arr.append([])
            var = []
            for k in range(len(concat_initial_dict[a])):
                var = [*concat_initial_dict[a][k], *var]
            arr[i] = var
            i += 1
        join_arr = []
        for a in range(len(arr)):
            join_arr.append(["".join(set(arr[a]))])
        new_string_init = ""
        for a in initial_states:
            new_string_init += str(a)
        new_initial_state = {new_string_init : dict(zip(symbols, join_arr))}
        return new_initial_state, new_string_init




    def check_term_stat(self, terminal_stat, CDFA):
        Arr_T_NT = list(CDFA)
        run = True
        for i in range(len(CDFA)):
            for j in range(len(CDFA[i])):
                y = 0
                while y in range(len(CDFA[i][j])) and run == True:
                    if CDFA[i][j][0][y] in terminal_stat:  # check if each states belongs to the terminal state
                        Arr_T_NT[i][j] = ["T"]
                        run = False
                    else:
                        Arr_T_NT[i][j] = ["NT"]
                    y += 1
                    run = True

                """while run == 1:
                    y = 0
                    while y in range(len(terminal_stat)) and run == 1:
                        chek = terminal_stat[y]
                        if CDFA[i][j][0] == chek:
                            Arr_T_NT[i][j] = ["T"]
                            run = 0
                        else:
                            Arr_T_NT[i][j] = ["NT"]
                        y += 1
                    run = 0"""
        return Arr_T_NT

    def T_NT_groups(self, new_states):
        T_NT_groups = [[], []]
        for i in new_states:
            if i.startswith("<=>"):
                T_NT_groups[0].append(i.replace("<=> ", ""))
            elif i.startswith("<--"):
                T_NT_groups[0].append(i.replace("<-- ", ""))
            else:
                i = i.replace("    ", "")
                i = i.replace("--> ", "")
                T_NT_groups[1].append(i)
        return T_NT_groups

    def T_NT_pattern(self, nb_states):
        pattern = ["NT", "T"]
        full_pattern = []
        for i in range(nb_states):
            full_pattern.append(pattern)
        return list(itertools.product(*full_pattern))


    def pattern_association(self, T_NT_pattern, T_NT_table):

        Arr = [[]]

        for i in range(len(T_NT_pattern)):
            for j in range(len(T_NT_table)):
                # here I take the different pattern possible of our table in terms of T and NT and I compare it to our actual T/NT table
                if T_NT_pattern[i] == T_NT_table[j]:
                    Arr[i].append(j)
                # When I compare them I add the number of which line as such a pattern

        return Arr

    def minimization(self, asso_arr, CDFA_table, NameFile):

        Arr_transition = []
        Arr_min = []


        file = open(NameFile, "r")
        line = ""
        for z in range(6):
            line = file.readline()

        # Now I will check if state in the same pattern have transition going in another pattern or in the same
        for i in range(len(asso_arr)):
            # We retrieve the emplacement of the states associate to the first pattern
            for j in range(len(asso_arr[i])):

                while line != "":
                    if line.startswith(str(j)):
                        Arr_transition.append(line[2])
                    line = file.readline()
                file.close()
                file = open(NameFile, "r")
                line = ""
                for z in range(6):
                    line = file.readline()

                    for x in range(len(Arr_transition)):
                        if Arr_transition[x] in asso_arr[i] == False:
                            Arr_min.append(j)
                            Arr_min.append(x)
                            del asso_arr[i][j]

                    Arr_transition.clear()

            for i in range(len(Arr_min)):
                if i % 2 != 0:
                    for j in range(len(asso_arr)):
                        if Arr_min[i] in asso_arr[j] == True:
                            Arr_min[i] = j
main = Main()
main.menu()
