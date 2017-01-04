# TaobaoRing Design Document

This document is written for architect, fronted & backend programmers, test engineers, etc. To help you understand the whole project targets and everything about website design details. It's not for visual design or user interface design, but focus on database, network, modules design. Programmers should follow this document to write codes.

## Overview of the project

Our target is design & development a stable, security, scalable, SEO friendly online e-commercial website for TaobaoRing. The codes should not be only workable, but also need to be clear, clean, efficient, and elegant.

We have a running website [here](https://www.taobaoring.com). Deployed in Linode VPS Servers. We built the site from Sept. 2011, it's 5 years now. As the time goes on, we're not satisfied with it's architecture and quality. We call it TaobaoRing 1.0, now we want a much better designed website 2.0.

Since we have 5 years' operation, there are lots real business data in our system. Meanwhile, our customers and employees are familiar with current pages layouts, so the compatibility is the first thing in design. The password, order history, shopping cart, payment records should be migrated to new website smoothly. The current website will be detailed introduced in [legacy system](#legacy-system) section. 

### System requirements and dependencies

#### 1. Basic requirements

- Final delopy system: CentOS 6.x
- Web server: Apache 2.x
    - Support HTTPS
    - Support HTTP/2
- Database: MySQL 5.6
    - Support master-master hot backup
    - Database daily backup
- Main develop language: PHP 5.6
    - Based on current codes
- JavaScript: jQuery 2.x
- Cache: Memecached?
- CDN: Cloudflare?
- System monitor(Disk/CPU/Network/Database, etc.)
- Log subsystem(We can check logs from network)

#### 2. Main 3rd part dependencies

- Email provider: Google Apps
- Language translator: Google Transalte
- Main dependent PHP libries
    - MaxMind: IP Library
- Barcode generate & decode
- Crawler library
- Page parser library
- Image processing library(Add watermark, etc.)

#### 3. API dependencies

**3.1 Basic APIs**

- Social media account to sign up with our website.
- Payments: PayPal API, etc.

**3.2 Advanced APIs**

We can discuss these advanced APIs later. Because it's hard to understand and not urgency for us.

- [Taobao API](https://open.taobao.com/) 
- [17Track API](http://www.17track.net/en)
- [4px API](http://www.4px.com/4px-en.html)

#### 4. Plugins

It's easy to deploy these plugins on our website. 

- LiveChat
- Addthis
- Google analytics

#### 5. Language

User pages should support both English & Chinese.

Administrator pages should support Chinese.

## Overview of the main functions

### Common functions

- [Order state transition diagram](#order-stat-stransition-diagram)
- [Database and tables](#database)
- [Email system](#email-system)
- [Email templates](#email-templates)
- [Ticket system](#ticket-system)
- [Full text search](#full-text-search)
- [CMS](#cms)
- [Notification system](#notification-system)
- [Omni Editor](#omni-editor)
- [TaobaoRing Coin](#coin-payment)
- [Coupon](#coupon)
- [Cache](#cache)
- [SKU](#sku)
- [Photo Management](#photo-management)
- [System Log](#system-log)
- [Translate](#translate)
- [MySQl](#mysql)

### User end pages

- [Sign in, Sign up, reset password, etc.](#sign-in)
- [Social media account to sign up](#social-account-sign-up)
- [User Center](#user-center)
- [Shopping cart](#shopping-cart)
- [Payment form](#payment-form)
- [Order list](#order-list)
- [History order list](#history-order-list)
- [FAQ](#faq)
- [News pages](#news)
- [Feedback pages](#feedback)
- [Search the website](#search)
- [Blog](#blog)

### Administrator pages

- [Admin order page](#admin-order)
- [Admin user page](#admin-user)
- [Admin feedback page](#admin-feedback)
- [Admin FAQ](#admin-faq)
- [Admin news page](#admin-news)
- [Admin employee management](#admin-employee-management)
- [Admin search](#admin-search)
- [Admin blog](#admin-blog)

## Details of each functions

<a name="order-stat-stransition-diagram"></a>

### Order state transition diagram

![](images/fourtek-order-status.png)

<a name="database"></a>

### Database and tables

`address` - Record user mailing address. Each user can save multiple mailing address, one of them can be set to default. `abandon` means user don't need this address any more.

`adminwords` - Record some our defined translation words, from English to Chinese, or from Chinese to English. 

`articles` - Core of CMS. Very simple, used to publish News or knowledge articles. [todo] Need to enhance/replace it, make it as a real CMS.

`charges` - Record our other fees or income outside of the order system. Like salaries expense, Taobao promote income etc. [todo] Need to enhance it.

`coin_account` - Record customer TaobaoRing coin account information. All customers have a `user` table record, but only some of them created coin account.

`coin_trades` - Record all coin trades. Remeber, we don't record balance in coin_account, we'll get it via coin trades table for real time. [todo] Need to enhance it, support partial payment, coupon, etc. And make the coin trade more safe.

`comments` - Record customer comments for `articles`. [todo] Need to enhance/replace it, should support upload pictures, use emoji, etc.

`daily_parcels` - We'll receive many parcels each day. When we receive a parcel, we'll use scanner to get the tracking number. Before the end of the working day, we'll upload all tracking numbers to database. Then, we'll know today's parcel count, and we can search with a tracking number to know if we received it or not.

`emails` - Store pending emails, refer the [GFW](#gfw-email) section. [todo] Enhance/replace it, only send emails, we need to retrieve emails from Gmail.

`feedback` - Record customer's feedback and our replies for feedback. [todo] Need to enhance it, should support upload pictures/video, use emoji, get coupon, etc.

`finance` - Record order's income, expense, tips/notice for ourselves. Each order have 1 finance record. [todo] Enhance it.

`gconf` - Global configure. Store API keys, etc. [todo] Enhance it.

`glog` - Global log. Store some important logs, we can check these logs via web pages. [todo] Enhance it.

`items` - Record Taobao item information. 1 order has multiple items records.

`item_balance` - Record refund/debt of a item. Only we need to refund customers(e.g. out of stock after customer paid) or customers debt us(e.g. add this item after they make the 1st payment), we'll create a item balance record.

`items_observe` - Record problem items need to track.

`message` - Record some important actions to notify. [todo] Enhance/replace it, create a new notify system.

`opnotes` - Discard, we'll delete it.

`orders` - Record customer orders. Refer [orders table](#orders-table).

`order_log` - Record logs of orders, we can check them via web pages. At the bottom of admin order page.

`order_msg` - Discard, we'll delete it.

`order_status` - Translate order status to human readable.

`payments` - Record payments of order. Each order have 2 payments. [todo] Enhance it, some order have 1, 2, or more than 2 payments.

`porder_shelves` - Record shelf code and it's human readable name.

`sellers` - Record our comment for sellers. [todo] Enhance it.

`sprice` - Record international shipping fee.

`stats_month_finance` - Record each month's income and expense. We'll know our finance situation from this table.

`tborders` - We can parse Taobao orders list, then convert Taobao orders to our order system orders.

`tborder_times` - We use this table to convert Taobao items list to our order system item list.

`tbparcels`, `tbparcel_orders`, `timezone`, `trackmap`, `tracksync`, ignore them now.

`tumblr` - Record Tumblr posts. [todo] We'll use our own CMS to replace it.

`users` - Record all registered users.

`user_info` - More about user information. 

`user_login` - Record user login actions.

<a name="orders-table"></a>

#### Orders Table

Set order\_id = 0 to delete order. 

`sid` means the full order id, it's a fixed 16 characters.

`addr_id` means each order should have a mailing address.

`weight` means the final weight of the parcel waiting for ship. [todo] Sometimes we have split a huge parcel to several small parcels, the the weight will be the total weight of all small parcels, we need to record each small parcels weight.

`domestic_delivering_fee` shipping fee from sellers to us.

`shipping_fee` means the final international shipping fee. [todo] Like `weight`, we should record each small parcels international shipping fee.

`status` means order status, refer [order state transition diagram](#order-stat-stransition-diagram).

`order_time` date time of the customer submit order.

`post_way` means international shipping method. [todo] Like `weight`, we should record each small parcels shipping method.

`post_number` mean international tracking number for parcel. [todo] Like `weight`, we should record each small parcels tracking number.

`post_time` means when we shipped the parcel. [todo] Like `weight`.

`wp_date` means when we set order to `wait_user_pay1`. Record this timestamp, then we can check recent unpaid orders and confirm with customers again.

`buy_time` means when we buy the order. Record this timestamp, we can track the bought orders and make sure we'll receive all parcels from sellers in time.

`track_time` means when we find we cannot receive all order parcels in time, we'll track it again.

`tracker` means who will take charge for track this order.

`memo` set by customer, their own tips for this order.

`user_msg` user message for this order. [todo] Replace it with a new/real message system.

`msg_reply` our replies for user message. [todo] Replace it with a new/real message system.

`service_fee` the service fee of this order: service fee = (items cost + domestic delivering fee) * commion rate of this order.

`total_price` means the the 1st payment: total price = (1)items cost + (2)domestic delivering fee + (3)service fee.

`items_cost` means sum of all items cost.

`qdate` means when we receive all items.

`tag` means the internal status of the order, just tips for ourselves.

`combine_orders` means customers want to ship several orders together. [todo] Enhance it, it's a string stored in each combined orders, not smart and easy to operate.

`limits` employees can set limits to this order. E.g. `forbid_paypal` means we don't accept PayPal for this order, `pay_once` means combine 1st & 2nd payment together, `deposit` means this is coin deposit order.

`confirmor` means which employees will take charge of this order's confirmation.

`buyer` means which employees will take charge of this order's purchase.

`assigned`, `modified`, please refer codes directly: `libs/types/admin_order.php` and `libs/types/order.php`.

`paypal_addr`, `paypal_info` record PayPal info, some order should be shipped to only PayPal address.

`decl` means we'll record the custom declare info. Refer codes. [todo] Need to enhance.

`shelf_label` means which shelf will hold on this order.

`psid` means parent order's full id. When we fork items from order to a new order, the new order will have `psid` field to record it's parent order id.

`pre_wt`, `pre_wts`, `pre_by`, `pre_ts` please refer `htdocs/admin/prew.php`.

<a name="email-trigger"></a>

### Email trigger

Email trigger described when and who will trigger an email. Each email links to it's templates.

#### Customers email trigger

- When customers sign up - [active_account](#email-active-account)
- When customers submit forgot their password - [password_reset](#email-password-reset)
- When customers submit change their password - [password_change](#email-password-change)
- When customers make payment:
    - Submit offline payment(Western Union, etc.) - [payment_pending](#email-payment-pending)
    - PayPal IPN - [payment_complete](#email-payment-complete)
    - PayPal eCheck - [payment_echeck](#email-payment-echeck)
    - PayPal eCheck Complete - [payment_echeck_complete](#email-payment-echeck-complete)
    - PayPal eCheck failed - [payment_echeck_failed](#email-payment-echeck-failed)
    
#### Administrator email trigger

- Waiting for purchase payment - [wait_payment_for_purchase](#email-wait-payment-for-purchase)
- Waiting for shipping payment - [wait_payment_for_ship](#email-wait-payment-for-ship)
- Confirm payment status
    - Offline payment complete - [payment_complete](#email-payment-complete)
    - Offline payment failed - [payment_failed](#email-payment-failed)
    - Offline payment delayed - [payment_delay](#email-payment-delay)
- Receive returned parcels from Chinese Custom - [return_domestic](#email-return-domestic)
- Receive returned parcels from overseas - [return_overseas](#email-return-overseas)
- Queries & communicate with customers about items
    - Help us to confirm - [item_confirm](#email-item-confirm)
    - Help us to purchase - [item_purchase](#email-item-purchase)
    - Help us to check - [item_check](#email-item-check)
    - Items are out of stock - [item_out_of_stock](#email-item-out-of-stock)
    - All items are out of stock - [order_out_of_stock](#email-order-out-of-stock)

<a name="ticket-system"></a>

### Ticket System

Ticket system is based on Email system. Core functions of ticket system:

- All customer emails(include new created and replied emails) will conver to ticket.
- Register customer can create/reply ticket via website page.
- Administrator can set rules for ticket.
    - Rules1: auto delete. If title, sender include some keywords, delete the ticket automatically.
    - Rules2: auto assign. If title, sender include some keywords, assign to an agent automatically.
- Private notes, only visible by agents.
- Auto reply during the holiday or special situation.
- Agents can add tags for tickets, just like normal email client tags.

#### Ticket Status

Ticket status:

- `New`: when customers send a new email to us, or fire a new ticket from website.
- `Pending`: we've assigned the tickets to an agent.
- `On hold`: we cannot handle it now, will deal it later.
- `Waiting for customers`: we've replied the tickets.
- `Closed`: we've resolved the tickets, or it's expired.
- `Deleted`: it's a spam or useless.

**When customer send a new email to us**

- The email will convert to a new ticket.
- Ticket status will be `New`.
- Customer will receive a new email include his original email contents, ticket id, and website ticket link.

**When customer fire a new ticket from website**

- Ticket status will be `New`.
- If customer select an associated order/item, and we know which employee will take charge of it, set status to `Pending`.
- Customer will receive a new email include his original email contents, ticket id, and website ticket link.

**When ticket assigned to an agent**

- Ticket status will be `Pending`.

**When agent reply ticket**

- Ticket status will be `Waiting for customers`.
- Customer will receive email include the reply contents, ticket id, and website ticket link.

**When customer reply ticket**

- Ticket status will be `Pending`.
- Customer will receive email include his replied contents, ticket id, and website ticket link.

**On hold status**

- If cannot handle the ticket immediately, agent will set it to `On hold`, means we stuck in something, will deal it later.
- Customer will not receive email.

**Closed status**

- Agents can close tickets actively.
- If tickets stay in `Waiting for customers` more than 48 hours, tickets will be closed automatically.
- Once the tickets closed, customers will receive email include the ticket status, ticket id, and website ticket link.
- Reply `Closed` ticket will reopen it and change back to `Pending`.
- Customer can repone ticket via website ticket page, and change back to `Pending`.

**Deleted status**

- Agents can delete tickets actively.
- We can set rules to delete tickets. Like title include some keywords, e.g.: `auto-reply`. 
- Customer will not receive email.

#### Fronted pages

**Ticket list page**

`Opened`, `Last Updated` means date time.

![](images/ticket-list.png)

**Create ticket page**

- `Regarding` is a selectable input:
    - `Consulting`
    - `Payment`
    - `Parcel Tracking`
    - `Combine or Split Orders`
    - `After Sales`
    - `Business Cooperation`
    - Make `Regarding` feilds editable in admin page(admin can create or delete `Regarding` list).
    - We can create filter rules for `Regarding`, e.g.: all `Payment` tickets will assigned to a agent automatically.
- If jump from `order` or `item`, record in database, show to customers and admin.
- The `Description` field, customer can upload pictures, photos, clipboard images(screen capture), etc. Just like Basecamp's input.

![](images/create-ticket.png)

**Ticket filter rules**

Default filter rule: if ticket include order_id, get `confirmor` from orders table, if `confirmor` is not None, assign this ticket to `confirmor`(agent).

We have these admin editable rules:

- if title include words.
- if email recipient equals.
- if email sender equals.
- if ticket regarding equals.

action:

- assign to agent.
- delete.

![](images/tickets-filter.png)

**Update ticket page**

![](images/update-ticket.png)

<a name="full-text-search"></a>

#### Backend pages

- htdocs/admin/ticket-list.php
    - htdocs/admin/ticket-list.php?agents=all 
    - htdocs/admin/ticket-list.php?agents=all&status=TICKET_STATUS 
    - htdocs/admin/ticket-list.php?agents=AGENT_NAME&status=TICKET_STATUS 
    - htdocs/admin/ticket-list.php?status=TICKET_STATUS 
    - htdocs/admin/ticket-list.php?agents=AGENT 
- htdocs/admin/ticket.php
    - htdocs/admin/ticket.php?ticket_id=TICKET_ID (check TICKET ID details)
- htdocs/admin/search.php
    - htdocs/admin/search.php?keyword=KEYWORD
    - htdocs/admin/search.php?keyword=KEYWORD&agent=AGENT_NAME

**All tickets list**

- The yellow box means we can select tickets
- All selected tickets can be assigned to other agents

Because we'll have huge closed/deleted tickets, so we can hide these two status from side bar.

![](images/tickets-admin-all.png)

**My tickets list**

- Default status is `Pending` and `On Hold`.
- If more than 30 tickets(configurable), paging.
- If tickets stay in `Waiting for customers` more than 72 hours, change to `Closed` automatically.

![](images/tickets-admin-mine.png)

**Reply Ticket**

![](images/tickets-admin-reply.png)

**Search Ticket**

![](images/tickets-admin-search.png)

### Full Text Search

We'll provide a full text search bar for customers. Customers can search:

- `Order`: when input order ID, jump to order detail page.
- `Transaction`: when input transaction ID, jump to order detail page include that transaction.
- `Item`: when input item keywords/URL, list all orders include the item.
- `Memo`, `Message`, `Ticket`: when input keywords, search with these fields.
- Other keywords: search the whole website text, to list all articles include the keywords.

About administrator search bar, refer the codes: `htdocs/admin/search.php` and `htdocs/admin-search-*`.

<a name="cms"></a>

### CMS(Content Management System)

We need a simple builtin CMS.

#### Backend pages

**Category List**

Core functions:

- Add new category.
- Delete category.
- Add new article to category.
- Sort the sequence of categories(Supported by AJAX, and record in database). 

![](images/cms-category-list.png)

**Create New Category**

Core functions:

- Set category name.
- Allowed users reply or not.
- Visualable for users or not. We have some articles only for employees.

![](images/cms-category-create.png)

**Articles in Category**

Core functions:

- Article List.
- Visit count will not include search engine crawler visit.

![](images/cms-admin-news-list.png)

**Create New Article**

Core functions:

- Multiple SEO meta.
- Rich format text editor.

![](images/cms-news-create.png)

#### Fronted pages

**Article List**

Core functions:

- Article List.

![](images/cms-news-list.png)

**Article Page**

Core functions:

- Users can click helpful or not.
- If category set to allowed user reply, insert [Disqus](http://www.disqus.com) plugin.

![](images/cms-news-page.png)

<a name="omni-editor"></a>

### Omni Editor

All the main inputs should support omni editor, just like Basecamp text input box. We'll use CKEditor?
    
Core functions:

- Paste images from clipboard
- Drag and drop images

<a name="photo-management"></a>

### Photo Management

Current photo mangement codes are mess and complicated. We need to rewrite this module. 

Refer the codes: `htdocs/admin-order.php` `do_upload_photo()` function, `libs/photo.php`, `htdocs/photo-manage.php`.

**Current problems**

- Admin upload photos to berlinix.com
    - tbring.com run a crontab script to sync photos from berlinix
    - taobaoring.com users will get photo links from tbring.com
- Photo delete is complicated
    - if we delete photos on berlinix.com, we'll sign in tbring.com to delete again
- Huge photos stored in berlinix.com and tbring.com
    - berlinix.com has 4.2 GB photos
    - tbring.com has 17 GB photos
    - why tbring.com has more photos? check the [history](#history) section

tbring.com crontab set:

    */10 * * * * rsync -avz bailing@berlinix.com:/www/tbring/htdocs/photo/ /www/tbring/htdocs/photo/ >> /home/bailing/backup/sync.log 2>/dev/null

`-a` means `-rlptgoD`, `-r`: recursive, `-l`: soft link, `-p`: keep permission, `-tgo`: keep time, group, owner, `-D`: keep device information. 

`-v` means verbose output, `-z` means compress.

So, if we delete photos in tbring.com, we have to delete them again in tbring.com.

![](images/photo-sync-old.png)

**Core functions**

- Batch upload photos, and distribute to orders.
- Admin can move photos from one order to another.
- Admin can delete photos from order.
- Handle the old photos.
- How to store photos? In file system? In database? In 3rd part cloud? 
- How to sync photos? Or only store in berlinix.com, taobaoring.com only get photo links?

**Current implement**

- We use camera to take photo.
- Take photo for barcode, then order products.
- After we take all order photos, transfer camera memory card photos to PC.
- We'll rename the photos.
- Upload to website.
    - PHP script will distribute photos to orders

Please read the codes to understand this process.

![](images/take-photo-old.png)

**New implement**

We'll keep current implement, enhance it, meanwhile, support a new solution.

- We'll use phone to take photo. (HTML5 can use phone camera directly)
- Scan the barcode, find the order ID, and jump to admin order page.
- Take photos for all products.
- Upload photos to admin order page. (Compress photos to 640px width)
- Then next order photos.

New solution is much easier than current.

![](images/take-photo-new.png)

<a name="mysql"></a>

### MySQL

Dump database schema without data:

    $ mysqldump -ubailing -p --no-data taobaoring > /tmp/taobaoring.sql

<a name="user-center"></a>

### User Center

Core functions:

- User avatar from Gravatar, and allowed user upload their photo.
- User can change email address.
- User can change password.
- User can check their coin account balance and details.
- User can check their basic information.

#### Avatar

We should allowed user upload avatar photo. 

Default avatar is extract from: https://en.gravatar.com/ 

The URL format is: http://www.gravatar.com/avatar/MD5-STRING

Refer code: `libs/identicon.php`.

<a name="shopping-cart"></a>

### Shopping cart

1\. Crawl pages from these sites:

- Taobao.com
- Tmall.com
- 1688.com
- jd.com (less important) 
     
2\. Parse pages, extract core information from pages:     

- Price
- Seller
- Shopname
- Images
- Options(color, size, etc.)
- URL(Simplify URL before store)

3\. Users can provide their requirements:

- Quantity of the item
- Queries/Remarks of the item
- Keep/Remove the shoe box

<a name="legacy-system"></a>

## Legacy system

<a name="history"></a>

### History of TaobaoRing Website

1\. First of all, we have a simple website like this:

![](images/gfw0.png)

2\. But GFW blocked the route for us. Our employees cannot access www.taobaoring.com

![](images/gfw1.png)

GFW is a monster, maybe block a IP, block a whole machine room, or block a domain. We use Linode VPS, but seems the whole machine room is blocked.

3\. We changed our network layout like this:

![](images/gfw2.png)

Why we split database from website server? Because I thought we may change website frequently, but stay database unchange.

4\. But someday when we weak up, we found out that we were blocked again.

![](images/gfw3.png)

5\. We change the network again, deployed a new VPS in Beijing. 

![](images/gfw4.png)

For some legacy reason, didn't remove www.tbring.com.

- Stored 17GB order photos.
- Sometimes www.taobaoring.com cannot crawling pages from Taobao, we have to make www.tbring.com provide crawling service for www.taobaoring.com.

<a name="gfw-email"></a>

Since we cannot access Gmail(Google APP), when employees want to send email from blog.berlinix.com, we have to store the email to database(Beijing). Then MySQL master-master hot backup will sync the emails to database(US). We set a crontab script to read pending emails, and send them to Gmail server on www.taobaoring.com(US) website. Maybe it's the simplest solution for GFW. Let me have a simple conclusion.

- blog.berlinix.com emails will store on database(beijing), and will be sent by www.taobaoring.com script.
- www.taobaoring.com emails can be sent to Gmail directly.
- [todo] We should retrieve emails from Gmail via IMAP, store on database(US), and sync to database(beijing).
- [todo] Our employees can handle emails on blog.berlinix.com directly, don't have to use VPN tools to access Gmail.

6\. My idea network layout is like this.

![](images/gfw5.png)
