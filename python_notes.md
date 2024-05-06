# configparser

    config.ini and config.conf are the conf fies used commonly where data is stored in sections of key value pairs. The strings varaibles in here needn't be enclosed in the quotes
    # Reading "C:\Users\sowja\OneDrive\Desktop\work\DE_notes\config.ini" and 
    # Reading "C:\Users\sowja\OneDrive\Desktop\work\DE_notes\config.conf"

    import configparser
    config = configparser.ConfigParser()
    config.read('C:\\Users\sowja\OneDrive\Desktop\work\DE_notes\config.conf')

    # using items 
    for k,v in config.items('default'):
        print(k, ': ', v)
        
    # using get method (section, key)
    config.get('default', 'name')

    # using section as arg, and providing a default value 
    config['default'].get('lastname', 'j')