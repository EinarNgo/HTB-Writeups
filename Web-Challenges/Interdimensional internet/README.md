# Interdimensional internet writeup
Challenge description: aw man, aw geez, my grandpa rick is passed out from all the drinking, where is a concentrated dark matter calculator when you need one, aw geez

## Looking around
When we visit the webpage, we see this.
![Website](website.png)

There is a number under the picture that seems to change everytime the webpage refresh.

In inspect element under sources, we can see there is a hint for a /debug page. When we visit the page, all the source code was there.

Source code:
```yml
    from flask import Flask, Response, session, render_template
    import functools, random, string, os, re

    app = Flask(__name__)
    app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY', 'tlci0GhK8n5A18K1GTx6KPwfYjuuftWw')

    def calc(recipe):
        global garage
        builtins, garage = {'__builtins__': None}, {}
        try: exec(recipe, builtins, garage)
        except: pass

    def GFW(func): # Great Firewall of the observable universe and it's infinite timelines
        @functools.wraps(func)
        def federation(*args, **kwargs):
            ingredient = session.get('ingredient', None)
            measurements = session.get('measurements', None)

            recipe = '%s = %s' % (ingredient, measurements)
            if ingredient and measurements and len(recipe) >= 20:
                regex = re.compile('|'.join(map(re.escape, ['[', '(', '_', '.'])))
                matches = regex.findall(recipe)
                
                if matches: 
                    return render_template('index.html', blacklisted='Morty you dumbass: ' + ', '.join(set(matches)))
                
                if len(recipe) > 300: 
                    return func(*args, **kwargs) # ionic defibulizer can't handle more bytes than that
                
                calc(recipe)
                # return render_template('index.html', calculations=garage[ingredient])
                return func(*args, **kwargs) # rick deterrent

            ingredient = session['ingredient'] = ''.join(random.choice(string.lowercase) for _ in xrange(10))
            measurements = session['measurements'] = ''.join(map(str, [random.randint(1, 69), random.choice(['+', '-', '*']), random.randint(1,69)]))

            calc('%s = %s' % (ingredient, measurements))
            return render_template('index.html', calculations=garage[ingredient])
        return federation

    @app.route('/')
    @GFW
    def index():
        return render_template('index.html')
    
    @app.route('/debug')
    def debug():
        return Response(open(__file__).read(), mimetype='text/plain')

    if __name__ == '__main__':
        app.run('0.0.0.0', port=1337)
```

## Understanding the code
One of the first line is a static SECRET_KEY, that means we can use it later to craft a flask session cookie.
```YML
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY', 'tlci0GhK8n5A18K1GTx6KPwfYjuuftWw')
```

We can see the app routes, / is the landing page and /debug is the page we are trying to understand the source code. This means there are no other page on the website. Only noteable point is the webapp calling a @GFW function. 
```YML
    @app.route('/')
    @GFW
    def index():
        return render_template('index.html')
    
    @app.route('/debug')
    def debug():
        return Response(open(__file__).read(), mimetype='text/plain')
```

The @GFW function acts as a firewall. The function get the values ingredients and measurements from the flask session cookie and converts the values them into a variable, and then check the length is between 20 and 300. The function also check if the variables contains a character that is blacklisted.
```YML
    def GFW(func): # Great Firewall of the observable universe and it's infinite timelines
        @functools.wraps(func)
        def federation(*args, **kwargs):
            ingredient = session.get('ingredient', None)
            measurements = session.get('measurements', None)

            recipe = '%s = %s' % (ingredient, measurements)
            if ingredient and measurements and len(recipe) >= 20:
                regex = re.compile('|'.join(map(re.escape, ['[', '(', '_', '.'])))
                matches = regex.findall(recipe)
                
                if matches: 
                    return render_template('index.html', blacklisted='Morty you dumbass: ' + ', '.join(set(matches)))
                
                if len(recipe) > 300: 
                    return func(*args, **kwargs) # ionic defibulizer can't handle more bytes than that
                
                calc(recipe)
                # return render_template('index.html', calculations=garage[ingredient])
                return func(*args, **kwargs) # rick deterrent

            ingredient = session['ingredient'] = ''.join(random.choice(string.lowercase) for _ in xrange(10))
            measurements = session['measurements'] = ''.join(map(str, [random.randint(1, 69), random.choice(['+', '-', '*']), random.randint(1,69)]))

            calc('%s = %s' % (ingredient, measurements))
            return render_template('index.html', calculations=garage[ingredient])
        return federation
```

Looking at the calc function, there is a pontentially vulnerability in exec. If we could work out how to pass values into the exec function we would be able to leak data on the system or potentially get a shell on server.
```YML
    def calc(recipe):
        global garage
        builtins, garage = {'__builtins__': None}, {}
        try: exec(recipe, builtins, garage)
        except: pass
```