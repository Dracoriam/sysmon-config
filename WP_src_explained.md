# Word Press

## Source Explained

### Σελίδα Login

#### Επιτρέπει την αυθεντικοποίηση των χρηστών

**Όνομα αρχείου**: `wp-login.php`

**Κομμάτι κώδικα**:

Μέσα στο αρχείο `wp-includes\user.php` ορίζεται η συνάρτηση `wp_signon()`:

```php
    wp_signon( array $credentials = array(), string|bool $secure_cookie = '' )
    /**
     * Authenticates and logs a user in with 'remember' capability.
     *
     * The credentials is an array that has 'user_login', 'user_password', and
     * 'remember' indices. If the credentials is not given, then the log in form
     * will be assumed and used if set.
     *
     * The various authentication cookies will be set by this function and will be
     * set for a longer period depending on if the 'remember' credential is set to
     * true.
     *
     * Note: wp_signon() doesn't handle setting the current user. This means that if the
     * function is called before the {@see 'init'} hook is fired, is_user_logged_in() will
     * evaluate as false until that point. If is_user_logged_in() is needed in conjunction
     * with wp_signon(), wp_set_current_user() should be called explicitly.
     *
     * @since 2.5.0
     *
     * @global string $auth_secure_cookie
     *
     * @param array       $credentials   Optional. User info in order to sign on.
     * @param string|bool $secure_cookie Optional. Whether to use secure cookie.
     * @return WP_User|WP_Error WP_User on success, WP_Error on failure.
     */
```

Μέσα στο `wp-login.php` χρησιμοποιείτε ως:

```php
    $user = wp_signon( array(), $secure_cookie );
```

**Εξήγηση πώς υλοποιείται**:

Η συναρτηση [wp_signon()](https://developer.wordpress.org/reference/functions/wp_signon/https://https://developer.wordpress.org/reference/functions/wp_signon/) είναι αυτη που κάνει πρακτικα τον log-in. H μεταβλητη `$credentials` είναι προαιρετική καθως αν δεν την παρέχουμε η συνάρτηση `wp-signon()` χρησιμοποιεί τη μεταβλητή `$_POST` για να αντλήσει της απαραίτητες πληροφορίες. Η διαδικασία είναι ότι η ιστοσελιδα θα στήσει ενα authentication cookie και μεσου αυτού θα γίνει η ταυτοποίηση.

### Σελίδα καταλόγου προϊόντων

#### Διαθέσιμη μόνο σε αυθεντικοποιημένους χρήστες (προϊόντων)

Όνομα αρχείου:
Κομμάτι κώδικα:
Πώς επιτυγχάνεται;;;;;

#### Κάθε προϊόν που παρουσιάζεται θα πρέπει να ακολουθείται από σύνδεσμο που το προσθέτει στο καλάθι αγορών [1]

**Όνομα αρχείου**: `wp-content\themes\blocksy\functions.php`

**Κομμάτι κώδικα**:

```php
add_action( 'template_redirect', 'redirect_if_user_not_logged_in' );

function redirect_if_user_not_logged_in(){
    $post = get_post();
    $allowed_pages_if_not_loged_in = array( 'welcome', 'my-acc' );
    if ( ! is_null($post) ) {
        if ( ! in_array($post->post_name, $allowed_pages_if_not_loged_in)) {
            if ( ! is_user_logged_in() ) {
                $url_acc = get_permalink( get_page_by_path( 'my-acc' ) );
                wp_redirect($url_acc); 
                exit;
            }
        }   
    }
}
```

**Πώς επιτυγχάνεται**:

Στο αρχείο `wp-content\themes\blocksy\functions.php` μπορούμε να προσθέσουμε συναρτήσεις και κώδικα για να πετυχουμε καποιες λειτουργιες. Για την επιτυχεις απομόνωση του site υλοποιήθηκε η συνάρτηση `redirect_if_user_not_logged_in()` η οποία κάνει hook στο action `'template_redirect'` και κάνει redirect ενα ο χρηστης δεν ειναι αυθεντικοποιημένος.

Το Documentation του WordPress μας ενημερώνει ότι το `do_action( 'template_redirect' )` εκτελείτε πριν αποφασιστεί ποιο template θα φορτωθεί. Εκεί είναι και η κατάλληλη θέση για να γίνει έλεγχος του χρήστη.

* Η μεταβλητη `$post` έχει πολλαπλές καταχωρίσεις μέσα της που αφορούν τη συγκεκριμένη σελιδα που ο χρήστης προσπαθει να φορτώσει.
  * Εαν `is_null($post)` η σελίδα δεν υπάρχει. Άρα δεν γίνεται redirect.
* Το πεδίο `$post->post_name` μας δίνει το slug της σελίδας που θέλουμε να φορτωσουμε. (Το slug οριζεται στο Dashboard/Pages)
  * Έαν αυτή δεν είναι στη λίστα με της επιτρεπτές σελιδες `$allowed_pages_if_not_loged_in` τοτε γίνεται redirect

#### Θα επιτρέπεται στο χρήστη αναζήτηση στον κατάλογο και τα αποτελέσματα της αναζήτησης θα εμφανίζονται στη σελίδα του καταλόγου [1]

**Όνομα αρχείου**: `wp-content\themes\blocksy\woocommerce\product-searchform.php`

**Κομμάτι κώδικα**:

```html
    <form   role="search"
            method="get" 
            class="woocommerce-product-search search-form" 
            action="<?php echo esc_url( home_url( '/' ) ); ?>" >
        <label  class="screen-reader-text" 
                for="woocommerce-product-search-field-<?php echo isset( $index ) ? absint( $index ) : 0; ?>" >
            <?php esc_html_e( 'Search for:', 'blocksy' ); ?>
        </label>

        <input  type="search" 
                id="woocommerce-product-search-field-<?php echo isset( $index ) ? absint( $index ) : 0; ?>" 
                class="search-field" 
                placeholder="<?php echo esc_attr__( 'Search products&hellip;', 'blocksy' ); ?>" 
                value="<?php echo get_search_query(); ?>" 
                name="s" />

        <button class="search-submit" 
                aria-label="<?php echo __('Search', 'blocksy')?>"
                >
            <svg    class="ct-icon" 
                    width="15" 
                    height="15" 
                    viewBox="0 0 15 15"
                    >
                <path d="M14.8,13.7L12,11c0.9-1.2,1.5-2.6,1.5-4.2c0-3.7-3-6.8-6.8-6.8S0,3,0,6.8s3,6.8,6.8,6.8c1.6,0,3.1-0.6,4.2-1.5l2.8,2.8c0.1,0.1,0.3,0.2,0.5,0.2s0.4-0.1,0.5-0.2C15.1,14.5,15.1,14,14.8,13.7z M1.5,6.8c0-2.9,2.4-5.2,5.2-5.2S12,3.9,12,6.8S9.6,12,6.8,12S1.5,9.6,1.5,6.8z"/>
            </svg>

            <span data-loader="circles"><span></span><span></span><span></span></span>
        </button>

        <input  type="hidden" 
                name="post_type" 
                value="product" />
</form>
```

**Πώς επιτυγχάνεται**:
Την αναζήτηση τον προϊόντων την διαχειρίζεται το e-commerce plugin *WooCommerce*. Συγκεκριμένα το *WooCommerce* παρέχει μια συλλογη με template files `wp-content\plugins\woocommerce\templates\` τα οποιά όμως πρακτικα δεν χρησιμοποιούνται απο το site. Ο εκάστοτε Theme Developer θα αντιγράψει το template στο Theme του και θα το επεξεργαστει εκεί. Αυτο γίνεται για λόγους ακεραιότητας του plugin. Στο οικοσυστημα του WordPress χρησιμοποιούνται hooks και templates ωστε να υπάρχει εν δυνάμη παντα ενα backup και να μην αλλοιώνεται ο κώδικας της σελίδας. Αυτο καθιστά το site *Modular*.

Άρα το αρχείο που διαχειρίζεται τη λειτουργία *Search* είναι το `wp-content\themes\blocksy\woocommerce\product-searchform.php` και όχι το `wp-content\plugins\woocommerce\templates\product-searchform.php`.

Οπως φαίνεται στο παραπάνω απόσπασμα κώδικα χρησιμοποιώντας HTML και php δημιουργεί ένα query.

Με inspection βλέπουμε το παρακάτω σαν αποτέλεσμα:

```html
    <form   role="search"
            method="get" 
            class="woocommerce-product-search search-form" 
            action="http://localhost/WP2/">
        <label  class="screen-reader-text" 
                for="woocommerce-product-search-field-0">Αναζήτηση για:
        </label>
        <input  type="search" 
                id="woocommerce-product-search-field-0" 
                class="search-field" 
                placeholder="Αναζήτηση προϊόντων…" 
                value="" 
                name="s">
        <button class="search-submit" aria-label="Αναζήτηση">
            <svg class="ct-icon" width="15" height="15" viewBox="0 0 15 15">
                <path d="M14.8,13.7L12,11c0.9-1.2,1.5-2.6,1.5-4.2c0-3.7-3-6.8-6.8-6.8S0,3,0,6.8s3,6.8,6.8,6.8c1.6,0,3.1-0.6,4.2-1.5l2.8,2.8c0.1,0.1,0.3,0.2,0.5,0.2s0.4-0.1,0.5-0.2C15.1,14.5,15.1,14,14.8,13.7z M1.5,6.8c0-2.9,2.4-5.2,5.2-5.2S12,3.9,12,6.8S9.6,12,6.8,12S1.5,9.6,1.5,6.8z"> </path>
            </svg>
            <span data-loader="circles">
                <span></span>
                <span></span>
                <span></span>
            </span>
        </button>
        <input type="hidden" name="post_type" value="product">
    </form>
```

Έχουμε σε HTML `<form method="get" >` άρα πρακτικα η φορμα αυτη, στην περιπτωση μας το search box θα κάνει αναζητηση δημιουργώντας ενα link. Πχ ενα αναζητήσουμε για την λέξη `'hoodie'` και τη φραση `'red hoodie'`θα παρουμε το link:

* http://localhost/WP2/?s=hoddie&post_type=product
* http://localhost/WP2/?s=red+hoddie&post_type=product

Προορισμός είναι το `action="http://localhost/WP2/"` και στέλνει δυο πληροφορίες, τα δύο `<input  >`, δηλαδή το Search Phrase που έβαλε ο χρήστης και το post_type που είναι `'hidden'` και ειναι παντα `'product'`

#### Η σελίδα θα αναφέρει αν παρουσιάζονται όλα τα προϊόντα ή μόνο τα προϊόντα που αφορούν μια συγκεκριμένη αναζήτηση

Όνομα αρχείου:
Κομμάτι κώδικα:
Πώς επιτυγχάνεται;;;;;

#### Η σελίδα θα πρέπει να φέρει υποσημείωση (footer) που θα περιγράφει τον αριθμό των αντικειμένων που έχουν προστεθεί στο καλάθι ή συνοπτική παρουσίαση των αντικειμένων που έχουν προστεθεί στο καλάθι

Όνομα αρχείου:
Κομμάτι κώδικα:
Πώς επιτυγχάνεται;;;;;

### Σελίδα πληρωμών

#### Διαθέσιμη μόνο σε αυθεντικοποιημένους χρήστες (πληρωμών)

Όνομα αρχείου:
Κομμάτι κώδικα:
Πώς επιτυγχάνεται;;;;;

#### Παρουσιάζει τα προϊόντα που έχουν προστεθεί στο καλάθι

Όνομα αρχείου:
Κομμάτι κώδικα:
Πώς επιτυγχάνεται;;;;;

#### Ο χρήστης θα μπορεί να συμπληρώσει τη διεύθυνση αποστολής σε φόρμα της σελίδας

Όνομα αρχείου:
Κομμάτι κώδικα:
Πώς επιτυγχάνεται;;;;;

#### Με την αποστολή των στοιχείων της φόρμας, θα παρουσιάζονται για τελική έγκριση όλα τα στοιχεία της παραγγελίας στο χρήστη (διεύθυνση, προϊόντα στο καλάθι) [1]

Όνομα αρχείου:
Κομμάτι κώδικα:
Πώς επιτυγχάνεται;;;;;

#### Αν ο χρήστης εγκρίνει την παραγγελία τότε αυτή αποστέλλεται με email προς το διαχειριστή του καταστήματος

Όνομα αρχείου:
Κομμάτι κώδικα:
Πώς επιτυγχάνεται;;;;;
