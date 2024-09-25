<script>
  import { LucidePlus, Trash2, ImageUp, RotateCcw, Printer } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '',
      created: '',
      due: '',
      from: '',
      fromTaxIdName: '',
      fromTaxId: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxIdName: '',
      toTaxId: '',
      toContact: { mail: '', phone: '' }
    },
    items: [{ desc: '', price: '', quantity: '' }],
    taxPercent: '',
    discountPercent: '',
    note: '',
    payment: {
      accountNumber: '',
      accountName: '',
      bank: '',
      ifsc: '',
      swiftCode: '',
      customField: ''
    },
    currency: 'USD'
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar' },
    { code: 'EUR', symbol: '€', name: 'Euro' },
    { code: 'GBP', symbol: '£', name: 'British Pound' },
    { code: 'JPY', symbol: '¥', name: 'Japanese Yen' },
    { code: 'AUD', symbol: 'A$', name: 'Australian Dollar' },
    { code: 'CAD', symbol: 'C$', name: 'Canadian Dollar' },
    { code: 'CHF', symbol: 'CHF', name: 'Swiss Franc' },
    { code: 'CNY', symbol: '¥', name: 'Chinese Yuan' },
    { code: 'INR', symbol: '₹', name: 'Indian Rupee' },
    { code: 'RUB', symbol: '₽', name: 'Russian Ruble' },
    { code: 'BRL', symbol: 'R$', name: 'Brazilian Real' },
    { code: 'ZAR', symbol: 'R', name: 'South African Rand' },
    { code: 'MXN', symbol: '$', name: 'Mexican Peso' },
    { code: 'NZD', symbol: 'NZ$', name: 'New Zealand Dollar' },
    { code: 'SGD', symbol: 'S$', name: 'Singapore Dollar' }
  ];

  function getCurrencySymbol(code) {
    const currency = currencyOptions.find((c) => c.code === code);
    return currency ? currency.symbol : '';
  }

  // Utils --------------------------------------------------------------

  function handleImageChange(event) {
    const file = event.target.files[0];

    if (
      file &&
      (file.type === 'image/png' || file.type === 'image/jpeg' || file.type === 'image/webp')
    ) {
      const reader = new FileReader();
      reader.onload = () => {
        appState.company.logo = reader.result;
        save();
      };
      reader.readAsDataURL(file);
    } else {
      alert('Please select a valid image (.png, .jpg, or .webp)');
    }
  }

  function save() {
    localStorage.setItem('invoiceData', JSON.stringify(appState));
  }

  function addItem() {
    if (itemDesc != '' && itemPrice > 0 && itemQty > 0) {
      appState.items = [...appState.items, { desc: itemDesc, price: itemPrice, quantity: itemQty }];

      itemDesc = '';
      itemPrice = 1;
      itemQty = 1;

      save();
      document.getElementById('descBox').focus();
    }
  }

  function deleteItem(index) {
    appState.items = appState.items.filter((_, i) => i !== index);
    save();
  }

  function reset() {
    appState = {
      company: { name: 'Wayne Enterprises', logo: '' },
      invoice: {
        number: '1277',
        created: 'May 24, 2024',
        due: 'June 24, 2024',
        from: '1007 Mountain Drive, Gotham City',
        fromTaxIdName: 'GSTIN',
        fromTaxId: '27AABCU9603R1ZM',
        fromContact: { mail: 'bruce@wayneenterprises.com', phone: '+1 555-0199' },
        to: '321 Infinite Loop, Dimension C-137',
        toTaxIdName: 'Interdimensional Tax ID',
        toTaxId: 'C-137-RS-123456',
        toContact: { mail: 'rick@c137.com', phone: '+1 555-0118' }
      },
      items: [
        { desc: 'Batmobile Repair', price: '15000', quantity: '1' },
        { desc: 'Grappling Hooks', price: '500', quantity: '20' },
        { desc: 'Interdimensional Portal Gun', price: '200000', quantity: '1' }
      ],
      taxPercent: '18',
      discountPercent: '10',
      note: 'Thank you for your business. Wubba Lubba Dub-Dub!',
      payment: {
        accountNumber: '9876 5432 1098',
        accountName: 'Wayne Enterprises',
        bank: 'Gotham National Bank',
        ifsc: 'GNB0001234',
        swiftCode: 'GNBKUS6S',
        customField: 'Payment due within 30 days'
      },
      currency: 'USD'
    };

    save();
  }

  // Startup ------------------------------------------------------------

  onMount(() => {
    const savedData = localStorage.getItem('invoiceData');
    if (savedData) {
      appState = JSON.parse(savedData);
    } else {
      reset();
    }
  });

  let itemDesc = '';
  let itemPrice = 1;
  let itemQty = 1;

  $: subTotal = appState.items.reduce((total, item) => {
    return total + item.price * item.quantity;
  }, 0);

  $: discountAmount = subTotal * (+appState.discountPercent || 0) / 100;
  $: taxableAmount = subTotal - discountAmount;
  $: taxAmount = taxableAmount * (+appState.taxPercent || 0) / 100;
  $: totalDue = taxableAmount + taxAmount;
</script>

<svelte:head>
  <title>Billie - No Strings Attached Invoice Generator</title>
  <meta
    name="description"
    content="Create and download invoices instantly, with zero signups or tracking."
  />
</svelte:head>

<svelte:body on:click={() => save()} />

<div
  class="print:rotate-0 print:m-0 max-w-screen-md mx-auto px-6 py-8 flex flex-col space-y-6 font-inter bg-white print:shadow-none shadow-lg"
>
  <!-- Company & Invoice Details --------------------------------------->
  <div class="flex flex-row justify-between">
    <div class="flex flex-col gap-4">
      <div>
        {#if appState.company.logo}
          <div class="flex flex-row items-center print:space-x-0 space-x-4">
            <div class="print:hidden flex flex-col border rounded-xl shadow-md">
              <button
                class="p-2"
                on:click={() => {
                  appState.company.logo = null;
                }}
              >
                <Trash2 />
              </button>
              <button class="p-2" on:click={() => document.getElementById('imageInput').click()}>
                <ImageUp />
                <input
                  id="imageInput"
                  type="file"
                  accept=".png,.jpg,.jpeg,.webp"
                  on:change={handleImageChange}
                  class="hidden"
                />
              </button>
            </div>
            <img
              src={appState.company.logo}
              alt="Company Logo"
              class="max-h-18 max-w-40 object-cover m-0"
            />
          </div>
        {:else}
          <button
            class="print:hidden p-2 flex flex-row gap-2 rounded-lg shadow-md border cursor-pointer"
            on:click={() => document.getElementById('imageInput').click()}
          >
            <ImageUp />
            <p>Click to select an image for logo</p>
            <input
              id="imageInput"
              type="file"
              accept=".png,.jpg,.jpeg,.webp"
              on:change={handleImageChange}
              class="hidden"
            />
          </button>
        {/if}
      </div>
      <input class="font-bold text-2xl border p-2" type="text" bind:value={appState.company.name} />
    </div>

    <div class="relative flex flex-col items-end gap-1">
      <h2 class="font-bold text-2xl text-[#1E6F5C] text-right">
        INVOICE :
        <input
          type="text"
          size="4"
          placeholder="2341"
          maxlength="4"
          bind:value={appState.invoice.number}
          class="border p-1"
        />
      </h2>
      <div class="flex flex-col space-y-2 absolute -right-20 print:hidden">
        <button
          on:click={() => {
            reset();
          }}
          class="p-2 rounded-lg bg-white group hover:bg-[#15273c] active:scale-90 transition-all duration-100 ease-in shadow"
        >
          <RotateCcw strokeWidth={1.5} class="group-hover:stroke-white" />
        </button>
      </div>
      <p class="print:hidden">
        Created :
        <input
          bind:value={appState.invoice.created}
          type="text"
          size="10"
          placeholder="May 24,2024"
          maxlength="13"
          class="border p-1"
        />
      </p>
      <p class="print:hidden">
        Due :
        <input
          bind:value={appState.invoice.due}
          type="text"
          size="10"
          placeholder="June 24,2024"
          maxlength="13"
          class="border p-1"
        />
      </p>
      <div class="print:hidden mt-2">
        <select bind:value={appState.currency} class="border p-1">
          {#each currencyOptions as option}
            <option value={option.code}>{option.code} - {option.name}</option>
          {/each}
        </select>
      </div>
    </div>
  </div>

  <hr />

  <!-- From & To  ------------------------------------------------------>

  <div class="flex flex-row justify-between">
    <div>
      <h4 class="mb-4 font-bold">From</h4>
      <textarea
        style="resize: none;"
        placeholder="Wayne Enterprises, 1007 Mountain Drive, Gotham City"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.from}
        class="border p-2"
      ></textarea>
      <div class="mt-2">
        <p>
          {appState.invoice.fromTaxIdName || 'Tax ID Name'} :
          <input
            type="text"
            bind:value={appState.invoice.fromTaxIdName}
            class="border p-1"
            placeholder="GSTIN"
          />
        </p>
        <p>
          {appState.invoice.fromTaxId || 'Tax ID'} :
          <input
            type="text"
            bind:value={appState.invoice.fromTaxId}
            class="border p-1"
            placeholder="27AABCU9603R1ZM"
          />
        </p>
        <p>
          Email :
          <input
            type="email"
            bind:value={appState.invoice.fromContact.mail}
            class="border p-1"
            placeholder="bruce@wayneenterprises.com"
          />
        </p>
        <p>
          Phone :
          <input
            type="text"
            bind:value={appState.invoice.fromContact.phone}
            class="border p-1"
            placeholder="+1 555-0199"
          />
        </p>
      </div>
    </div>
    <div>
      <h4 class="mb-4 font-bold">To</h4>
      <textarea
        style="resize: none;"
        placeholder="Rick Sanchez, 321 Infinite Loop, Dimension C-137"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.to}
        class="border p-2"
      ></textarea>
      <div class="mt-2">
        <p>
          {appState.invoice.toTaxIdName || 'Tax ID Name'} :
          <input
            type="text"
            bind:value={appState.invoice.toTaxIdName}
            class="border p-1"
            placeholder="Interdimensional Tax ID"
          />
        </p>
        <p>
          {appState.invoice.toTaxId || 'Tax ID'} :
          <input
            type="text"
            bind:value={appState.invoice.toTaxId}
            class="border p-1"
            placeholder="C-137-RS-123456"
          />
        </p>
        <p>
          Email :
          <input
            type="email"
            bind:value={appState.invoice.toContact.mail}
            class="border p-1"
            placeholder="rick@c137.com"
          />
        </p>
        <p>
          Phone :
          <input
            type="text"
            bind:value={appState.invoice.toContact.phone}
            class="border p-1"
            placeholder="+1 555-0118"
          />
        </p>
      </div>
    </div>
  </div>

  <!-- New Item Form --------------------------------------------------->

  <form class="print:hidden flex flex-row justify-between gap-2">
    <button
      disabled={itemDesc == ''}
      on:click={() => {
        addItem();
      }}
      class="absolute -ml-20 print:hidden rounded-full p-2 bg-white shadow cursor-pointer active:scale-90 transition-all duration-100 ease-in"
    >
      <LucidePlus stroke="#557571" />
    </button>

    <input
      id="descBox"
      type="text"
      bind:value={itemDesc}
      placeholder="Item description"
      class="border rounded-l p-2 grow"
      on:keypress={(e) => {
        e.key == 'Enter' && addItem();
      }}
    />
    <input
      type="number"
      bind:value={itemPrice}
      placeholder="Unit Price"
      min="1"
      class="border p-2 w-32"
      on:keypress={(e) => {
        e.key == 'Enter' && addItem();
      }}
    />
    <input
      type="number"
      bind:value={itemQty}
      placeholder="Quantity"
      min="1"
      step="1"
      class="border p-2 w-32"
      on:keypress={(e) => {
        e.key == 'Enter' && addItem();
      }}
    />
    <p class="border p-2 w-32">
      {getCurrencySymbol(appState.currency)}{itemPrice * itemQty}
    </p>
  </form>

  <!-- Table ----------------------------------------------------------->

  <div class="flex flex-col">
    <div class="flex flex-row bg-[#1e6f5c]">
      <p class="font-bold text-white p-2 border grow break-all">Item Description</p>
      <input
        disabled
        class="p-2 bg-inherit border placeholder:font-bold placeholder:text-white"
        maxlength="7"
        size="6"
        type="text"
        placeholder="Unit Price"
      />
      <input
        disabled
        class="p-2 bg-inherit border placeholder:font-bold placeholder:text-white"
        maxlength="4"
        size="4"
        type="text"
        placeholder="Qty"
      />
      <input
        disabled
        class="p-2 bg-inherit border placeholder:font-bold placeholder:text-white"
        maxlength="4"
        size="12"
        type="text"
        placeholder="Total"
      />
    </div>
    {#each appState.items as item, index}
      <div class="flex flex-row even:bg-gray-50 break-all">
        <button
          on:click={() => {
            deleteItem(index);
          }}
          class="absolute -ml-20 print:hidden rounded-full p-2 bg-white shadow cursor-pointer active:scale-90 transition-all duration-100 ease-in"
        >
          <Trash2 color="#C96868" />
        </button>

        <p contenteditable="true" class="p-2 border grow break-all">{item.desc}</p>
        <input class="p-2 border" maxlength="7" size="6" type="text" bind:value={item.price} />
        <input class="p-2 border" maxlength="4" size="4" type="text" bind:value={item.quantity} />
        <input
          disabled
          class="p-2 border"
          maxlength="4"
          size="12"
          type="text"
          placeholder={`${getCurrencySymbol(appState.currency)}${item.price * item.quantity}`}
        />
      </div>
    {/each}

    <div class="flex flex-row even:bg-gray-50 break-all">
      <p class="p-2 border grow break-all text-right font-bold">Subtotal</p>
      <input
        disabled
        class="p-2 border"
        maxlength="2"
        size="12"
        type="text"
        placeholder={`${getCurrencySymbol(appState.currency)}${subTotal}`}
      />
    </div>
    <div class="flex flex-row even:bg-gray-50 break-all">
      <p class="p-2 border grow break-all text-right font-bold">Discount %</p>
      <input
        class="p-2 border"
        maxlength="2"
        size="12"
        type="text"
        bind:value={appState.discountPercent}
      />
    </div>
    <div class="flex flex-row even:bg-gray-50 break-all">
      <p class="p-2 border grow break-all text-right font-bold">Taxable Amount</p>
      <input
        disabled
        class="p-2 border"
        maxlength="2"
        size="12"
        type="text"
        placeholder={`${getCurrencySymbol(appState.currency)}${taxableAmount}`}
      />
    </div>
    <div class="flex flex-row even:bg-gray-50 break-all">
      <p class="p-2 border grow break-all text-right font-bold">Tax %</p>
      <input
        class="p-2 border"
        maxlength="2"
        size="12"
        type="text"
        bind:value={appState.taxPercent}
      />
    </div>
    <div class="flex flex-row even:bg-gray-50 break-all">
      <p class="p-2 border grow break-all text-right font-bold">Total Due</p>
      <input
        disabled
        class="p-2 border"
        maxlength="2"
        size="12"
        type="text"
        placeholder={`${getCurrencySymbol(appState.currency)}${totalDue}`}
      />
    </div>
  </div>

  <!-- Additional Details  --------------------------------------------->

  <div>
    <h4 class="mb-4 font-bold">Note</h4>
    <textarea
      bind:value={appState.note}
      style="resize: none;"
      placeholder="Thank you for your business. Wubba Lubba Dub-Dub!"
      cols="30"
      rows="3"
      maxlength="150"
      class="border p-2"
    ></textarea>
  </div>
  <p>Thank you for your business</p>
  <hr />

  <div class="flex flex-row justify-between break-inside-avoid">
    <div>
      <h4 class="mb-4 font-bold">Payment Info</h4>
      <ul>
        <li class="m-1 flex flex-row gap-1">
          <p>Account</p>
          <span>:</span>
          <input
            type="text"
            maxlength="20"
            placeholder="987654321098"
            bind:value={appState.payment.accountNumber}
            class="border p-1"
          />
        </li>
        <li class="m-1 flex flex-row gap-1">
          <p>A/C Name</p>
          <span>:</span>
          <input
            type="text"
            minlength="0"
            maxlength="28"
            placeholder="Wayne Enterprises"
            bind:value={appState.payment.accountName}
            class="border p-1"
          />
        </li>
        <li class="m-1 flex flex-row gap-1">
          <p>Bank Name</p>
          <span>:</span>
          <input
            type="text"
            minlength="0"
            maxlength="28"
            placeholder="Gotham National Bank"
            bind:value={appState.payment.bank}
            class="border p-1"
          />
        </li>
        <li class="m-1 flex flex-row gap-1">
          <p>IFSC</p>
          <span>:</span>
          <input
            type="text"
            maxlength="11"
            placeholder="GNB0001234"
            bind:value={appState.payment.ifsc}
            class="border p-1"
          />
        </li>
        <li class="m-1 flex flex-row gap-1">
          <p>SWIFT Code</p>
          <span>:</span>
          <input
            type="text"
            maxlength="11"
            placeholder="GNBKUS6S"
            bind:value={appState.payment.swiftCode}
            class="border p-1"
          />
        </li>
        <li class="m-1 flex flex-row gap-1">
          <p>Note</p>
          <span>:</span>
          <input
            type="text"
            maxlength="50"
            placeholder="Payment due within 30 days"
            bind:value={appState.payment.customField}
            class="border p-1"
          />
        </li>
      </ul>
    </div>
  </div>

  <!-- Download Button -------------------------------------------------->

  <div class="mt-6 flex justify-center print:hidden">
    <button
      on:click={() => {
        window.print();
      }}
      class="px-6 py-3 rounded-full bg-gradient-to-r from-blue-500 to-green-500 text-white font-bold shadow-lg transform hover:scale-105 transition-transform"
    >
      Download Invoice
    </button>
  </div>
</div>
