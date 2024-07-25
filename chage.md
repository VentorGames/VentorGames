import re
import os

class ChageInterpreter:
    def __init__(self):
        self.variables = {}
        self.functions = {}
        self.code = []

    def tokenize(self, code):
        return re.findall(r'\b\w+\b|[+\-*/()=<>!]+|"[^"]*"|[\n]', code)

    def parse_expression(self, tokens):
        if len(tokens) == 1:
            token = tokens[0]
            if token.isdigit():
                return int(token)
            elif token.replace('.', '', 1).isdigit():
                return float(token)
            elif token in self.variables:
                return self.variables[token]
            elif token.startswith('"') and token.endswith('"'):
                return token[1:-1]
            else:
                raise ValueError(f"Unknown token: {token}")

        if len(tokens) == 3:
            left = self.parse_expression([tokens[0]])
            right = self.parse_expression([tokens[2]])
            if tokens[1] == '+':
                return left + right
            elif tokens[1] == '-':
                return left - right
            elif tokens[1] == '*':
                return left * right
            elif tokens[1] == '/':
                return left / right
            elif tokens[1] == '>':
                return left > right
            elif tokens[1] == '<':
                return left < right
            elif tokens[1] == '==':
                return left == right
            elif tokens[1] == '!=':
                return left != right
            elif tokens[1] == '>=':
                return left >= right
            elif tokens[1] == '<=':
                return left <= right

        raise ValueError(f"Invalid expression: {' '.join(tokens)}")

    def save_project(self, project_name):
        filename = f"{project_name}.chage"
        with open(filename, 'w') as file:
            file.write('\n'.join(self.code))
        print(f"Project saved as {filename}")

    def load_project(self, filename):
        if os.path.exists(filename):
            with open(filename, 'r') as file:
                self.code = file.read().splitlines()
            print(f"Loaded project from {filename}")
            self.execute('\n'.join(self.code))
        else:
            print(f"File {filename} not found.")

    def execute(self, code):
        tokens = self.tokenize(code)
        i = 0
        while i < len(tokens):
            token = tokens[i]

            if token == 'print':
                i += 1
                expr_tokens = []
                while i < len(tokens) and tokens[i] != '\n':
                    expr_tokens.append(tokens[i])
                    i += 1
                result = self.parse_expression(expr_tokens)
                print(result)

            elif token == 'if':
                i += 1
                condition_tokens = []
                while i < len(tokens) and tokens[i] != '\n':
                    condition_tokens.append(tokens[i])
                    i += 1
                condition = self.parse_expression(condition_tokens)
                if condition:
                    i += 1
                    while i < len(tokens) and tokens[i] != 'end':
                        expr_tokens = []
                        while i < len(tokens) and tokens[i] != '\n':
                            expr_tokens.append(tokens[i])
                            i += 1
                        self.execute(' '.join(expr_tokens))
                        i += 1
                else:
                    while i < len(tokens) and tokens[i] != 'end':
                        i += 1

            elif token == 'loop':
                i += 1
                condition_tokens = []
                while i < len(tokens) and tokens[i] != '\n':
                    condition_tokens.append(tokens[i])
                    i += 1
                loop_start = i
                while self.parse_expression(condition_tokens):
                    i = loop_start
                    while i < len(tokens) and tokens[i] != 'end':
                        expr_tokens = []
                        while i < len(tokens) and tokens[i] != '\n':
                            expr_tokens.append(tokens[i])
                            i += 1
                        self.execute(' '.join(expr_tokens))
                        i += 1

            elif token == 'func':
                i += 1
                func_name = tokens[i]
                i += 1
                param_tokens = []
                while i < len(tokens) and tokens[i] != '\n':
                    param_tokens.append(tokens[i])
                    i += 1
                func_body_start = i + 1
                func_body_tokens = []
                while i < len(tokens) and tokens[i] != 'end':
                    func_body_tokens.append(tokens[i])
                    i += 1
                self.functions[func_name] = (param_tokens, func_body_tokens)

            elif token in self.functions:
                func_name = token
                i += 1
                arg_tokens = []
                while i < len(tokens) and tokens[i] != '\n':
                    arg_tokens.append(tokens[i])
                    i += 1
                param_tokens, func_body_tokens = self.functions[func_name]
                local_vars = {}
                for param, arg in zip(param_tokens, arg_tokens):
                    local_vars[param] = self.parse_expression([arg])
                old_vars = self.variables
                self.variables = local_vars
                self.execute(' '.join(func_body_tokens))
                self.variables = old_vars

            elif '=' in token:
                var_name = tokens[i-1]
                i += 1
                expr_tokens = []
                while i < len(tokens) and tokens[i] != '\n':
                    expr_tokens.append(tokens[i])
                    i += 1
                value = self.parse_expression(expr_tokens)
                self.variables[var_name] = value

            elif token == 'save':
                i += 1
                project_name = tokens[i]
                self.save_project(project_name)
                i += 1
                continue

            elif token == 'load':
                i += 1
                filename = tokens[i]
                self.load_project(filename)
                i += 1
                continue

            i += 1

print("Chage Programming Language")
print("Enter your code (type 'run' to execute, 'save <name>' to save, or 'load <filename>' to load and run a file):")

interpreter = ChageInterpreter()

while True:
    line = input().strip()
    if line.lower() == 'run':
        interpreter.execute('\n'.join(interpreter.code))
    elif line.lower().startswith('save '):
        project_name = line.split()[1]
        interpreter.save_project(project_name)
    elif line.lower().startswith('load '):
        filename = line.split()[1]
        interpreter.load_project(filename)
    elif line.lower() == 'exit':
        break
    else:
        interpreter.code.append(line)
