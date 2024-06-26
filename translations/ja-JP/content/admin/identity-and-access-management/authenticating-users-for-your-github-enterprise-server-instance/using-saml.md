---
title: SAMLの利用
redirect_from:
  - /enterprise/admin/articles/configuring-saml-authentication
  - /enterprise/admin/articles/about-saml-authentication
  - /enterprise/admin/user-management/using-saml
  - /enterprise/admin/authentication/using-saml
  - /admin/authentication/using-saml
  - /enterprise/admin/authentication/authenticating-users-for-your-github-enterprise-server-instance/using-saml
intro: 'You can configure SAML single sign-on (SSO) for {% data variables.product.product_name %}, which allows users to authenticate through a SAML identity provider (IdP) to access your instance.'
versions:
  ghes: '*'
type: how_to
topics:
  - Accounts
  - Authentication
  - Enterprise
  - Identity
  - SSO
---

## About SAML for {% data variables.product.product_name %}

SAML SSO allows people to authenticate and access {% data variables.product.product_location %} through an external system for identity management.

SAML は認証と認可のための XML ベースの標準です。 When you configure SAML for {% data variables.product.product_location %}, the external system for authentication is called an identity provider (IdP). Your instance acts as a SAML service provider (SP). For more information, see [Security Assertion Markup Language](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) on Wikipedia.

{% data reusables.enterprise_user_management.built-in-authentication %}

## サポートされているSAMLサービス

{% data reusables.saml.saml-supported-idps %}

{% ifversion ghes > 3.3 %}

If your IdP supports encrypted assertions, you can configure encrypted assertions on {% data variables.product.product_name %} for increased security during the authentication process.

{% endif %}

{% data reusables.saml.saml-single-logout-not-supported %}

## SAMLでのユーザ名についての考慮

各{% data variables.product.prodname_ghe_server %}ユーザ名は、SAMLの応答で次のアサーションのいずれかによって決定され、優先順位で並べられます。

- カスタムユーザ名属性 (定義済みかつ存在する場合)
- `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`アサーション (存在する場合)
- `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`アサーション (存在する場合)
- `NameID`要素

`NameID`要素は、他の属性が存在する場合でも必須です。

`NameID` と {% data variables.product.prodname_ghe_server %} ユーザ名の間にマッピングが作成されるため、`NameID` は永続的で一意でなければならず、ユーザのライフサイクルにおいて変更にさらされないようにする必要があります。

{% note %}

**注釈**: ユーザの `NameID` が IdP で変更された場合、ユーザが {% data variables.product.prodname_ghe_server %} インスタンスにサインインしようとすると、エラーメッセージが表示されます。 {% ifversion ghes %}To restore the user's access, you'll need to update the user account's `NameID` mapping. 詳しい情報については、「[ユーザの SAML `NameID` を更新する](#updating-a-users-saml-nameid)」を参照してください。{% else %} 詳しい情報については、「[エラー: '別のユーザーがすでにアカウントを所有しています'](#error-another-user-already-owns-the-account)」を参照してください。{% endif %}

{% endnote %}

{% data reusables.enterprise_management_console.username_normalization %}

{% data reusables.enterprise_management_console.username_normalization_sample %}

{% data reusables.enterprise_user_management.two_factor_auth_header %}
{% data reusables.enterprise_user_management.external_auth_disables_2fa %}

## SAMLのメタデータ

The service provider metadata for {% data variables.product.product_location %} is available at `http(s)://[hostname]/saml/metadata`.

アイデンティティプロバイダを手動で設定するなら、Assertion Consumer Service (ACS) URLは`http(s)://[hostname]/saml/consume`です。 これは`urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST`バインディングを利用します。

## SAMLの属性

以下の属性が利用できます。 You can change the attribute names in the [management console](/enterprise/{{ currentVersion }}/admin/guides/installation/accessing-the-management-console/), with the exception of the `administrator` attribute.

| デフォルトの属性名       | 種類 | 説明                                                                                                                          |
| --------------- | -- | --------------------------------------------------------------------------------------------------------------------------- |
| `NameID`        | 必須 | 永続ユーザ識別子。 任意の名前識別子の形式を使用できます。 どの代替アサーションも指定しない場合、{% data variables.product.prodname_ghe_server %}ユーザ名には`NameID`要素が使用されます。 |
| `administrator` | 任意 | この値が 'true' であれば、ユーザは自動的に管理者に昇格します。 他の値、あるいは値が存在しない場合は、ユーザは通常のユーザアカウントに降格します。                                               |
| `ユーザ名`          | 任意 | {% data variables.product.prodname_ghe_server %} のユーザ名                                                                    |
| `full_name`     | 任意 | ユーザのプロフィールページに表示されるユーザ名です。 ユーザはプロビジョニング後に名前を変更できます。                                                                         |
| `emails`        | 任意 | ユーザのメールアドレス。 複数指定することができます。                                                                                                 |
| `public_keys`   | 任意 | ユーザの公開 SSH キー。 複数指定することができます。                                                                                               |
| `gpg_keys`      | 任意 | ユーザの GPG キー。 複数指定することができます。                                                                                                 |

To specify more than one value for an attribute, use multiple `<saml2:AttributeValue>` elements.

```
<saml2:Attribute FriendlyName="public_keys" Name="urn:oid:1.2.840.113549.1.1.1" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
    <saml2:AttributeValue>ssh-rsa LONG KEY</saml2:AttributeValue>
    <saml2:AttributeValue>ssh-rsa LONG KEY 2</saml2:AttributeValue>
</saml2:Attribute>
```

## SAMLの設定

{% data reusables.enterprise_site_admin_settings.access-settings %}
{% data reusables.enterprise_site_admin_settings.management-console %}
{% data reusables.enterprise_management_console.authentication %}
1. **SAML**を選択してください。

   ![Screenshot of option to enable SAML authentication in management console](/assets/images/enterprise/management-console/auth-select-saml.png)
1. {% data reusables.enterprise_user_management.built-in-authentication-option %}

   ![Screenshot of option to enable built-in authentication outside of SAML IdP](/assets/images/enterprise/management-console/saml-built-in-authentication.png)
1. オプションで、未承諾応答SSOを有効化する場合は [**IdP initiated SSO**] を選択します。 デフォルトでは、{% data variables.product.prodname_ghe_server %}は未承認アイデンティティプロバイダ (IdP) 起点のリクエストに対して、IdPへの`AuthnRequest`返信で応答します。

   ![Screenshot of option to enable IdP-initiated unsolicited response](/assets/images/enterprise/management-console/saml-idp-sso.png)

   {% tip %}

   **Note**: We recommend keeping this value **unselected**. You should enable this feature **only** in the rare instance that your SAML implementation does not support service provider initiated SSO, and when advised by {% data variables.contact.enterprise_support %}.

   {% endtip %}

1. {% data variables.product.product_location %} 上のユーザの管理者権限を SAML プロバイダに決めさせたく**ない**場合、[**Disable administrator demotion/promotion**] を選択します。

   ![Screenshot of option to enable option to respect the "administrator" attribute from the IdP to enable or disable administrative rights](/assets/images/enterprise/management-console/disable-admin-demotion-promotion.png)
1. Optionally, to allow {% data variables.product.product_location %} to send and receive encrypted assertions to and from your SAML IdP, select **Require encrypted assertions**. For more information, see "[Enabling encrypted assertions](#enabling-encrypted-assertions)."

   ![Screenshot of "Enable encrypted assertions" checkbox within management console's "Authentication" section](/assets/images/help/saml/management-console-enable-encrypted-assertions.png)

   {% warning %}

   **Warning**: Incorrectly configuring encrypted assertions can cause all authentication to {% data variables.product.product_location %} to fail.

   - You must ensure that your IdP supports encrypted assertions and that the encryption and key transport methods in the management console match the values configured on your IdP. You must also provide {% data variables.product.product_location %}'s public certificate to your IdP. For more information, see "[Enabling encrypted assertions](#enabling-encrypted-assertions)."

   - Before enabling encrypted assertions, {% data variables.product.company_short %} recommends testing encrypted assertions in a staging environment, and confirming that SAML authentication functions as you expect. 詳しい情報については "[ステージングインスタンスのセットアップ](/admin/installation/setting-up-a-github-enterprise-server-instance/setting-up-a-staging-instance)"を参照してください。

   {% endwarning %}
1. In the **Single sign-on URL** field, type the HTTP or HTTPS endpoint on your IdP for single sign-on requests. この値はIdpの設定によって決まります。 If the host is only available from your internal network, you may need to [configure {% data variables.product.product_location %} to use internal nameservers](/enterprise/{{ currentVersion }}/admin/guides/installation/configuring-dns-nameservers/).

   ![Screenshot of text field for single sign-on URL](/assets/images/enterprise/management-console/saml-single-sign-url.png)
1. Optionally, in the **Issuer** field, type your SAML issuer's name. This verifies the authenticity of messages sent to {% data variables.product.product_location %}.

   ![Screenshot of text field for SAML issuer URL](/assets/images/enterprise/management-console/saml-issuer.png)
1. In the **Signature Method** and **Digest Method** drop-down menus, choose the hashing algorithm used by your SAML issuer to verify the integrity of the requests from {% data variables.product.product_location %}. Specify the format with the **Name Identifier Format** drop-down menu.

   ![Screenshot of drop-down menus to select signature and digest method](/assets/images/enterprise/management-console/saml-method.png)
1. [**Verification certificate**] の下で、[**Choose File**] をクリックし、IdP からの SAML のレスポンスを検証するための証明書を選択してください。

   ![Screenshot of button for uploading validation certificate from IdP](/assets/images/enterprise/management-console/saml-verification-cert.png)
1. Modify the SAML attribute names to match your IdP if needed, or accept the default names.

   ![Screenshot of fields for entering additional SAML attributes](/assets/images/enterprise/management-console/saml-attributes.png)

{% ifversion ghes > 3.3 %}

## Enabling encrypted assertions

To enable encrypted assertions, your SAML IdP must also support encrypted assertions. You must provide {% data variables.product.product_location %}'s public certificate to your IdP, and configure encryption settings that match your IdP.

{% warning %}

**Warning**: Incorrectly configuring encrypted assertions can cause all authentication to {% data variables.product.product_location %} to fail. {% data variables.product.company_short %} strongly recommends testing your SAML configuration in a staging environment. For more information about staging instances, see "[Setting up a staging instance](/admin/installation/setting-up-a-github-enterprise-server-instance/setting-up-a-staging-instance)."

{% endwarning %}

1. Configure SAML for {% data variables.product.product_location %}. For more information, see "[Configuring SAML settings](#configuring-saml-settings)."
{% data reusables.enterprise_installation.ssh-into-instance %}
1. Run the following command to output {% data variables.product.product_location %}'s public certificate.
   
        openssl pkcs12 -in /data/user/common/saml-sp.p12 -nokeys -passin pass:
1. In the output, copy the text beginning with `-----BEGIN CERTIFICATE-----` and ending with `-----END CERTIFICATE-----`, and paste the output into a plaintext file.
1. Sign into your SAML IdP as an administrator.
1. In the application for {% data variables.product.product_location %}, enable encrypted assertions.
   - Note the encryption method and key transport method.
   - Provide the public certificate from step 3.
{% data reusables.enterprise_site_admin_settings.access-settings %}
{% data reusables.enterprise_site_admin_settings.management-console %}
{% data reusables.enterprise_management_console.authentication %}
1. Select **Require encrypted assertions**.

   ![Screenshot of "Enable encrypted assertions" checkbox within management console's "Authentication" section](/assets/images/help/saml/management-console-enable-encrypted-assertions.png)
1. To the right of "Encryption Method", select the encryption method for your IdP from step 5.

   ![Screenshot of "Encryption Method" for encrypted assertions](/assets/images/help/saml/management-console-encrypted-assertions-encryption-method.png)
1. To the right of "Key Transport Method", select the key transport method for your IdP from step 5.

   ![Screenshot of "Key Transport Method" for encrypted assertions](/assets/images/help/saml/management-console-encrypted-assertions-key-transport-method.png)
1. **Save settings（設定の保存）**をクリックしてください。
{% data reusables.enterprise_site_admin_settings.wait-for-configuration-run %}

{% endif %}

## Updating a user's SAML `NameID`

{% data reusables.enterprise_site_admin_settings.access-settings %}
2. **SAML**を選択してください。 ![サイト管理者設定の "All users" サイドバー項目](/assets/images/enterprise/site-admin-settings/all-users.png)
3. ユーザーのリストで、`NameID` マッピングを更新するユーザ名をクリックします。 ![インスタンスユーザアカウントのリストにあるユーザ名](/assets/images/enterprise/site-admin-settings/all-users-click-username.png)
{% data reusables.enterprise_site_admin_settings.security-tab %}
5. [Update SAML NameID] の右にある [**Edit**] アイコンをクリックします。 ![SAML認証](/assets/images/enterprise/site-admin-settings/update-saml-nameid-edit.png)
6. [NameID] フィールドに、ユーザの新しい `NameID` を入力します。 ![入力済みの NameID を含むモーダルダイアログの "NameID" フィールド](/assets/images/enterprise/site-admin-settings/update-saml-nameid-field-in-modal.png)
7. [**Update NameID**] をクリックします。 ![モーダル内の更新された NameID 値の下の "Update NameID" ボタン](/assets/images/enterprise/site-admin-settings/update-saml-nameid-update.png)

## {% data variables.product.product_location %}へのアクセスの削除

アイデンティティプロバイダからユーザを削除したなら、そのユーザを手動でサスペンドもしなければなりません。 そうしなければ、そのユーザはアクセストークンあるいはSSHキーを使って引き続き認証を受けることができてしまいます。 詳しい情報については[ユーザのサスペンドとサスペンドの解除](/enterprise/admin/guides/user-management/suspending-and-unsuspending-users)を参照してください。

## レスポンスメッセージについての要求

レスポンスメッセージは以下の要求を満たさなければなりません。

- `<Destination>`要素はルートレスポンスドキュメントで指定されていなければならず、ACS URLに一致する必要があります。ただし、これはルートレスポンスドキュメントに署名がある場合のみです。 アサーションに署名がある場合は無視されます。
- `<AudienceRestriction>`要素の一部として、`<Audience>`要素は常に指定する必要があります。 It must match the `EntityId` for {% data variables.product.prodname_ghe_server %}. これは、`https://ghe.corp.example.com`というような、{% data variables.product.prodname_ghe_server %}インスタンスへのURLです。
- Each assertion in the response **must** be protected by a digital signature. これは、個々の`<Assertion>`要素に署名するか、`<Response>`要素を署名するかすることによって行います。
- `<Subject>`要素の一部として`<NameID>`要素を指定する必要があります。 任意の名前識別子の形式を使用できます。
- `Recipient` 属性は存在しなければならず、ACS URL に設定されなければなりません。 例:

```xml
<samlp:Response ...>
  <saml:Assertion ...>
    <saml:Subject>
      <saml:NameID ...>...</saml:NameID>
      <saml:SubjectConfirmation ...>
        <saml:SubjectConfirmationData Recipient="https://ghe.corp.example.com/saml/consume" .../>
      </saml:SubjectConfirmation>
    </saml:Subject>
    <saml:AttributeStatement>
      <saml:Attribute FriendlyName="USERNAME-ATTRIBUTE" ...>
        <saml:AttributeValue>monalisa</saml:AttributeValue>
      </saml:Attribute>
    </saml:AttributeStatement>
  </saml:Assertion>
</samlp:Response>
```

## SAML認証

{% data variables.product.prodname_ghe_server %} は、認証ログの _/var/log/github/auth.log_ で失敗した SAML 認証のエラーメッセージをログに記録します。 SAML レスポンス要件の詳細については、「[レスポンスメッセージの要件](#response-message-requirements)」を参照してください。

### エラー:「別のユーザがすでにアカウントを所有しています」

ユーザが SAML 認証を使用して初めて {% data variables.product.prodname_ghe_server %} にサインインすると、{% data variables.product.prodname_ghe_server %} はインスタンスにユーザアカウントを作成し、SAML `NameID` をアカウントにマップします。

ユーザが再度サインインすると、{% data variables.product.prodname_ghe_server %} はアカウントの `NameID` マッピングを IdP のレスポンスと比較します。 IdP のレスポンスの `NameID` が、{% data variables.product.prodname_ghe_server %} がユーザに対して想定している `NameID` とマッチしなくなると、サインインは失敗します。 ユーザには次のメッセージが表示されます。

> 別のユーザが既にアカウントを所有しています。 管理者に認証ログを確認するようご依頼ください。

このメッセージは通常、その人のユーザ名またはメールアドレスが IdP で変更されたということを示します。 {% ifversion ghes %}Ensure that the `NameID` mapping for the user account on {% data variables.product.prodname_ghe_server %} matches the user's `NameID` on your IdP. 詳しい情報については、「[ユーザの SAML `NameID` の更新](#updating-a-users-saml-nameid)」を参照してください。{% else %} `NameID` マッピングの更新については、{% data variables.contact.contact_ent_support %} にお問い合わせください。{% endif %}

### SAMLレスポンスが署名されていなかった場合、あるいは署名が内容とマッチしなかった場合、authログに以下のエラーメッセージが残されます。

If the `Recipient` does not match the ACS URL for your {% data variables.product.prodname_ghe_server %} instance, one of the following two error messages will appear in the authentication log when a user attempts to authenticate.

```
Recipient in the SAML response must not be blank.
```

```
Recipient in the SAML response was not valid.
```

IdP の `Recipient` の値を、{% data variables.product.prodname_ghe_server %} インスタンスの完全な ACS URL に設定してください。 For example, `https://ghe.corp.example.com/saml/consume`.

### エラー:「SAML レスポンスが署名されていないか、変更されています」

IdP が SAML レスポンスに署名しない場合、または署名が内容と一致しない場合、次のエラーメッセージが認証ログに表示されます。

```
SAML Response is not signed or has been modified.
```

IdP で {% data variables.product.prodname_ghe_server %} アプリケーションの署名済みアサーションを設定していることを確認してください。

### エラー:「Audience が無効です」または「アサーションが見つかりません」

IdP のレスポンスに `Audience` の値がないか、または正しくない場合、次のエラーメッセージが認証ログに表示されます。

```shell
Audience is invalid. Audience attribute does not match https://<em>YOUR-INSTANCE-URL</em>
```

IdP の `Audience` の値を、{% data variables.product.prodname_ghe_server %} インスタンスの `EntityId` に設定してください。これは、{% data variables.product.prodname_ghe_server %} インスタンスへの完全な URL です。 For example, `https://ghe.corp.example.com`.
