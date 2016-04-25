# Tutorial 05 Use of Data bag to store data
------
The data bag is a global variable that is stored as JSON data and is accessible from a Chef Server. A data bag is indexed for something and can be loaded by a recipe or accessed during a search.

### Pre-requisite
1. [Chocolatey](https://chocolatey.org)
2. Powershell
3. OpenSSL

### Instructions to create data bag
The current directory structure for the data bag will be as followed:
- path_to_chef_repo
  - chef-repo
    - .chef
    - cookbooks
      - cookbook1
      - cookbook2
      - ...
    - data_bags
      - data_bag_name
        - data_bag_item.json
        - ...
      - ...


1. Type the following command: `knife data bag create DATA_BAG_NAME DATA_BAG_ITEM`. eg. `knife data bag create config admin`.
2. To update, type the following command: `knife data bag from file DATA_BAG_NAME DATA_BAG_ITEM`. e.g `knife data bag from file config admin.json`, `knife data bag from file config C:\path_to_chef_repo\chef-repo\data_bags\config\admin.json`.
3. You can view your stored data bag in the Chef Manage user interface. Go to Policy > Data Bags.

### Instructions to create encrypted data bag
1. Under `knife.rb` config, put `knife[:editor] = "Notepad"` and `data_bag_encrypt_version   2`.
2. If you do not have Chocolatey installed, please install it as you will need it to install OpenSSL.
3. Open the Powershell and type the following the command: `openssl rand -base64 512`
4. Copy and paste the output into a text editor. Remember to remove all the next line before saving it as `encrypted_data_bag_secret`.
5. Place the `encrypted_data_bag_secret` in `C:\chef`.
6. Type the command `knife data bag create password mysql --secret-file C:\chef\encrypted_data_bag_secret`. ![alt text](https://github.com/ktlim86/chef-tutorial/blob/master/tutorial05/img/editor.JPG)

7. A editor will pop out. Fill in the `user` and the `pass`. Make sure the syntax must follow JSON convention. Save and exit. Below is the output. ![alt text](https://github.com/ktlim86/chef-tutorial/blob/master/tutorial05/img/output.JPG)

8. Type in the following command. `knife data bag show password my_sql`. You will see the `user` and `pass` are encrypted. ![alt text](https://github.com/ktlim86/chef-tutorial/blob/master/tutorial05/img/show_pass.JPG)

9. Type in the following command. `knife data bag show password my_sql --secret-file C:\chef\encrypted_data_bag_secret`. You will see the `user` and `pass` are decrypted. ![alt text](https://github.com/ktlim86/chef-tutorial/blob/master/tutorial05/img/show_pass2.JPG)

### Reference
1. https://docs.chef.io/data_bags.html
2. https://docs.chef.io/config_rb_knife.html
