################################################################################
# Copyright (c) 2017, 2018 IBM Corp. and others
#
# This program and the accompanying materials are made available under
# the terms of the Eclipse Public License 2.0 which accompanies this
# distribution and is available at https://www.eclipse.org/legal/epl-2.0/
# or the Apache License, Version 2.0 which accompanies this distribution and
# is available at https://www.apache.org/licenses/LICENSE-2.0.
#
# This Source Code may also be made available under the following
# Secondary Licenses when the conditions for such availability set
# forth in the Eclipse Public License, v. 2.0 are satisfied: GNU
# General Public License, version 2 with the GNU Classpath
# Exception [1] and GNU General Public License, version 2 with the
# OpenJDK Assembly Exception [2].
#
# [1] https://www.gnu.org/software/classpath/license.html
# [2] http://openjdk.java.net/legal/assembly-exception.html
#
# SPDX-License-Identifier: EPL-2.0 OR Apache-2.0 OR GPL-2.0 WITH Classpath-exception-2.0 OR LicenseRef-GPL-2.0 WITH Assembly-exception
################################################################################

omr_add_tracegen(j9vm.tdf)
add_library(j9vm SHARED
	annsup.c
	AsyncMessageHandler.cpp
	bchelper.c
	bindnatv.cpp
	BytecodeInterpreter.cpp
	callin.cpp
	classallocation.c
	ClassInitialization.cpp
	classloadersearch.c
	classseg.c
	classsupport.c
	createramclass.cpp
	DebugBytecodeInterpreter.cpp
	description.c
	dllsup.c
	drophelp.c
	exceptiondescribe.c
	exceptionsupport.c
	FastJNI_com_ibm_oti_vm_VM.cpp
	FastJNI.cpp
	FastJNI_java_lang_Class.cpp
	FastJNI_java_lang_ClassLoader.cpp
	FastJNI_java_lang_invoke_MethodHandle.cpp
	FastJNI_java_lang_J9VMInternals.cpp
	FastJNI_java_lang_Object.cpp
	FastJNI_java_lang_reflect_Array.cpp
	FastJNI_java_lang_ref_Reference.cpp
	FastJNI_java_lang_String.cpp
	FastJNI_java_lang_System.cpp
	FastJNI_java_lang_Thread.cpp
	FastJNI_java_lang_Throwable.cpp
	FastJNI_sun_misc_Unsafe.cpp
	findmethod.c
	gphandle.c
	growstack.cpp
	guardedstorage.c
	hookableAsync.c
	initsendtarget.cpp
	#Only on z/OS?
	#inlineleconditionhandler.c
	intfunc.c
	J9OMRHelpers.cpp
	javaPriority.c
	jnicgen.c
	jnicsup.cpp
	jnifield.cpp
	jniinv.c
	jnimem.c
	jnimisc.cpp
	jnireflect.cpp
	jvmfree.c
	jvminit.c
	jvminitcommon.c
	jvmrisup.c
	KeyHashTable.c
	leconditionexceptionsup.c
	linearswalk.c
	lockwordconfig.c
	logsupport.c
	lookuphelper.c
	lookupmethod.c
	MHInterpreter.cpp
	ModularityHashTables.c
	monhelpers.c
	montable.c
	NativeHelpers.cpp
	ObjectFieldInfo.cpp
	ObjectMonitor.cpp
	OutOfLineINL_com_ibm_jit_JITHelpers.cpp
	OutOfLineINL_java_lang_invoke_NativeMethodHandle.cpp
	OutOfLineINL_jdk_internal_misc_Unsafe.cpp
	ownedmonitors.c
	profilingbc.c
	rasdump.c
	rastrace.c
	resolvefield.cpp
	resolvesupport.cpp
	romclasses.c
	romutil.c
	segment.c
	StackDumper.c
	statistics.c
	stringhelpers.cpp
	swalk.c
	threadhelp.cpp
	threadpark.c
	throwexception.c
	visible.c
	VMAccess.cpp
	vmbootlib.c
	vmhook.c
	vmifunc.c
	vmizip.c
	vmphases.c
	vmprops.c
	vmruntimestate.c
	vmthinit.c
	vmthread.c
	xcheck.c

	${CMAKE_CURRENT_BINARY_DIR}/ut_j9vm.c
)

#hacks for M4 stuff
#TODO we should be detecting m4 properly
#also we probably need to fix up the path dir separators for windows
if(OMR_ARCH_X86)
	add_custom_command( OUTPUT xcinterp.s
		DEPENDS xcinterp.m4 run_constgen
		COMMAND m4 -I ${CMAKE_CURRENT_SOURCE_DIR}/../oti ${CMAKE_CURRENT_SOURCE_DIR}/xcinterp.m4 > xcinterp.s
	)
	add_custom_command(OUTPUT stackswap.s
		DEPENDS xa64/stackswap.m4 run_constgen
		COMMAND m4 -I ${CMAKE_CURRENT_SOURCE_DIR}/../oti ${CMAKE_CURRENT_SOURCE_DIR}/xa64/stackswap.m4 > stackswap.s
	)
	target_sources(j9vm
		PRIVATE
		xa64/unsafeHelper.s
		xcinterp.s
		stackswap.s
	)
elseif(OMR_ARCH_POWER) #TODO check linux and 64bit
	add_custom_command( OUTPUT pcinterp.s
		DEPENDS pcinterp.m4 run_constgen
		COMMAND m4 -I ${CMAKE_CURRENT_SOURCE_DIR}/../oti ${CMAKE_CURRENT_SOURCE_DIR}/pcinterp.m4 > pcinterp.s
	)
	target_sources(j9vm
		PRIVATE
			xl64/unsafeHelper.s
			pcinterp.s
	)
endif()

target_include_directories(j9vm
	PRIVATE
		${j9vm_SOURCE_DIR}/shared_common/include
		${CMAKE_CURRENT_BINARY_DIR}
)

target_link_libraries(j9vm
	PRIVATE
		j9vm_interface
		j9vm_gc_includes

		omrcore
		j9verutil
		j9thr
		j9util
		j9utilcore
		j9avl
		j9hashtable
		j9pool
		j9stackmap
		j9trc
		j9hookable
		j9prt
		j9bcv
		j9dyn
		j9simplepool
		j9zip
		ffi

		# link hacks
		# TODO: figure out why CMake needs these, but UMA does not
		j9util
		j9simplepool
		j9verutil
		j9stackmap
		j9bcv
		j9dyn
		j9util
		j9thr
)

install(
	TARGETS j9vm
	LIBRARY DESTINATION ${j9vm_SOURCE_DIR}
)
